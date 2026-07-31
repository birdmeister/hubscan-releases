# Creating the HubSpot Service Key for HubScan

HubScan reads a client portal through a **Service Key**: a read-only credential
you create once per portal. This guide covers where it lives, which scopes to
tick, the scopes you will not find (and should stop looking for), and what each
error actually means.

Nothing here writes to the portal. Every scope below is a read scope.

## 1. Where the key lives

In the client portal: **Settings -> Integrations -> Service Keys -> Create
service key**.

Two credentials are easy to confuse with it. Neither is a substitute:

- **Private Apps** (Settings -> Integrations -> Private Apps) is a different
  menu and a different credential. A Private App token has the same
  `pat-eu1-...` / `pat-na1-...` shape as a Service Key, so you cannot tell them
  apart by looking at the value.
- **The HubSpot CLI personal access key** (`hs api`, `~/.hscli/config.yml`)
  carries its own scope set, which does not include `automation`. Handy for
  poking at an endpoint, useless as a HubScan token.

Creating a Service Key needs **super admin** rights in the client portal. If you
do not have them, ask the client's HubSpot admin for a Service Key named
"HubScan (read-only audit)" with the scopes in section 2, and have them send you
the token value. It only reads: no data leaves your machine, and the client can
delete the key at any time to cut off access.

## 2. The scopes to tick

On the **Scopes** tab, search each name and tick it. All 15 are read-only and
all 15 are findable:

| Scope | What HubScan reads with it |
| --- | --- |
| `crm.objects.contacts.read` | Contacts created per user, ownership hygiene, marketing-contact bloat |
| `crm.objects.companies.read` | Company hygiene counts (no owner, stale, missing fields) |
| `crm.objects.deals.read` | Deal ownership and deal-hygiene counts |
| `tickets` | Ticket hygiene counts |
| `crm.objects.owners.read` | The owner list, including archived owners (how deactivated users are found) |
| `crm.objects.users.read` | The user list and their assigned seats |
| `crm.objects.forecasts.read` | Forecast submissions, so forecasting managers do not read as inactive |
| `crm.schemas.custom.read` | Which custom objects exist |
| `crm.objects.custom.read` | Records in those custom objects |
| `settings.users.read` | The portal's user list, super-admin flags and teams: the access-drift pillar |
| `automation.sequences.read` | Sequences per user |
| `automation` | Workflows, for automation health and the agent-readiness write map |
| `account-info.security.read` | Audit logs: last login per user, and portal tier |
| `scheduler.meetings.meeting-link.read` | Meeting links, split personal vs round-robin |
| `content` | Marketing email send volume, for the new-pricing credit estimate |

Two of these trip people up:

- **`tickets` is the whole scope name.** There is no `crm.objects.tickets.read`;
  the granular form does not exist for tickets.
- **`automation` is required on top of `automation.sequences.read`.** They look
  redundant and are not: the Workflows API rejects the sequences scope outright.
  Without plain `automation` the workflow scan goes dark, and HubScan can then
  recommend downgrading the seat of a user who is the fixed sender on a live
  sequence-enrolling workflow, which breaks that workflow.

Both are checked by `probe`, so a miss is caught before you scan.

Tick the scopes, create the key, and copy the **access token** into HubScan.
Professional and Enterprise portals grant all 15; a Professional portal simply
has less behind some of them. A couple are tied to what the portal owns, so a
portal without Sales Hub Professional cannot grant the sequences scope at all,
and HubScan will report that signal as unavailable.

## 3. The scopes you will not find

**`crm.objects.tasks.read`, `crm.objects.notes.read` and
`crm.objects.calls.read` have no toggle.** They are not hidden, not tier-gated,
and not something support can switch on: HubSpot exposes no checkbox for them at
all, in the Service Key UI or the Private App UI. The reads work anyway, through
an implicit grant, and HubScan measures tasks, notes and calls on portals whose
key never listed them.

So: do not go looking, and do not ask the client's admin to find them. Earlier
versions of this guide (and the app's own token guide) listed all three, which
sent people hunting for checkboxes that do not exist. That was our error.

The scope *names* still show up in HubSpot's 403 responses and therefore in
HubScan's warnings, because that is what HubSpot calls them. If you ever see one,
section 5 has the fix, and it is not "grant the scope".

## 4. Check the key before you scan

```bash
export HUBSPOT_TOKEN=<the-service-key>
./hubscan probe --portal=clientname
```

`probe` walks every endpoint the audit depends on and names any scope that is
genuinely missing. Fix, re-run, and only then scan. A scope you add later takes
effect as soon as you click **Update changes** in HubSpot: the token value stays
valid, no need to reissue or re-paste it.

## 5. When something 403s

Read the body, not just the status. HubSpot returns 403 for three different
situations and only one of them is about scopes.

**`MISSING_SCOPES`, with a scope name in
`errors[].context.requiredGranularScopes`.** A real missing scope. Tick that
exact name, click Update changes, re-run.

**A 403 that does not say `MISSING_SCOPES`.** Typically on meeting links:
*"User does not have permissions on this portal"*. This reads like an identity
problem and is neither identity nor scopes: **reissue the Service Key**. A key's
service-account identity is fixed when it is created, so a key whose scopes were
edited afterwards can be left in a stale state that no amount of scope-fiddling
clears. Verified live: reissuing the key made the same call return 200 with
nothing else changed. Do this before investigating anything else.

**Audit logs 403 on a Professional portal.** Expected, and not a scope problem.
Audit logs are Enterprise-only, so the endpoint refuses regardless of what the
key holds. HubScan uses exactly that refusal to infer the portal tier. Confusing
detail worth knowing: the response body says *"This app hasn't been granted all
required scopes"*, naming scopes for what is really a subscription gate. A
Professional portal is supposed to answer 403 here.

**A tasks, notes or calls 403.** See section 3: there is no scope to grant.
Reissue the key. If it survives a reissue, contact HubSpot support and quote the
scope name from the response.

## 6. What a missing scope costs you

HubScan degrades rather than fails. A signal it cannot read is marked
unavailable in the report, with a line explaining what was skipped and how that
skews the recommendations: users whose work lives in an unmeasured object can
look inactive and draw a false downgrade suggestion.

Two consequences worth flagging to a client who wants to withhold a scope:

- Without `automation`, sequence senders are unprotected. That is the one
  degrade that can turn a HubScan recommendation into a broken workflow.
- Without `content`, the new-pricing sidecar loses real send volume and falls
  back to a figure you enter by hand.

## 7. Removing access

Deleting HubScan from your machine does not disable anything on the client's
side. To cut off access, delete the key in the portal: **Settings ->
Integrations -> Service Keys -> delete the HubScan key**. Do this per portal.

Then delete that portal's entry from your own settings (`~/.hubscan/portals.toml`,
or the `.hubscan` folder in your user folder on Windows), and the reports you
generated, which hold client data.

## Support

Stuck on a key or a 403 that does not match anything here: **info@hubscan.nl**.
