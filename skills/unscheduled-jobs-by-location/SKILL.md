---
name: unscheduled-jobs-by-location
description: >
  Use this skill when the user asks about unassigned or unscheduled jobs for a specific
  client, especially when they want to see where those jobs are located or grouped
  geographically. Trigger phrases include: "unscheduled jobs for", "unassigned jobs for",
  "what jobs haven't been scheduled for", "jobs with no tech for", "open dispatch queue
  for", "what's not assigned for", "show me jobs that need scheduling for", or any request
  combining a client name with the idea of jobs that are pending assignment or scheduling.
  Always use this skill (rather than portal-resources or query-db) when the user wants
  unassigned/unscheduled jobs grouped or sorted by location.
metadata:
  version: "0.1.0"
---

When this skill triggers, retrieve unassigned or unscheduled work orders for a specific
client and present them grouped by geographic proximity.

## Step 1 — Confirm the client

If the user named a client, proceed. If not, ask: "Which client would you like to check?"

Look up the client ID:
```sql
SELECT client_id, company
FROM client
WHERE company LIKE '%<name>%'
```
Use `client_id` and `company` — those are the correct column names. The `client` table has
no `id` or `name` column.

If multiple clients match, list them and ask the user to confirm which one.

## Step 2 — Fetch unassigned/unscheduled work orders

Use the `workorders_unassigned_or_unscheduled` tool with:
- `client_id`: the ID found in Step 1
- `team_id`: 0 (all teams)

This returns all work orders where statcode ≤ 60 and either no installer is assigned or
no install start date is set.

Note the `id` of every returned work order — you'll need them in Step 3.

## Step 3 — Enrich with location data

Fetch address and coordinates for all returned work orders in a single query. Use this
exact join pattern — **do not deviate**. The naive approaches (`site.addr_id`, direct
column guesses) do not work; this is the query that does:

```sql
SELECT
  wo.id,
  wo.summary,
  wo.statcode,
  s.company   AS site_name,
  s.siteid,
  a.address1,
  a.city,
  st.abbr     AS state,
  a.zip,
  a.lat,
  a.lng
FROM workorder wo
LEFT JOIN site    s  ON wo.site_id     = s.id
LEFT JOIN address a  ON a.entity_id   = s.id
                     AND a.address_type = 'site'
LEFT JOIN state   st ON a.state       = st.id
WHERE wo.id IN (<comma-separated list of WO ids>)
```

**Critical schema notes:**
- `workorder.id` — the primary key is `id`, not `workorder_id`
- `site.id` — use `id`, not `site_id`
- Address-to-site join: `address.entity_id = site.id AND address.address_type = 'site'`
  — `site.addr_id` is always 0 and must NOT be used
- `address` columns: `address1`, `city`, `state` (int FK), `zip`, `lat`, `lng`
- `state` table: join on `address.state = state.id`, use `state.abbr` for abbreviation
- `site.company` — the store name; `site.siteid` — the client's internal store number

## Step 4 — Group by geographic proximity

Use the Haversine formula to calculate the distance in miles between every pair of job
sites, then cluster jobs within roughly 75 miles of each other into the same group.

**Haversine formula (miles):**
```
d = 3958.8 × 2 × arcsin(√(
      sin²((lat2−lat1)/2) +
      cos(lat1) × cos(lat2) × sin²((lng2−lng1)/2)
    ))
```
(all angles in radians)

Clustering approach: pick the job with the most neighbors within 75 miles as the cluster
anchor, assign all within-range jobs to that cluster, then repeat for remaining unassigned
jobs. Label each cluster by its anchor's city and state (e.g., "Minnetonka, MN area").

If any work orders have no coordinates (lat = 0 and lng = 0), list them in a separate
"No location data" section at the bottom rather than silently omitting them.

## Step 5 — Present results

For each geographic cluster, show:
- A header: **[City, State] area — N job(s)**
- A table with columns: WO #, Summary, Site Name, Site ID, Address, Status

Use the statcode reference to translate numeric statcodes to human-readable labels:

| statcode | Label |
|---|---|
| 22 | Dispatched (no tech) |
| 26 | Open (unassigned) |
| 40 | Assigned |
| 50 | Accepted |
| 55 | Incomplete |
| 60 | Scheduled (no date) |

At the bottom, note the total count and flag any jobs where statcode is 50 with a
scheduled date that has already passed — these had a technician but no confirmed install
date and may be overdue.

## Follow-up offers

After presenting results, offer:
- "Want me to find available technicians near any of these clusters?"
- "Should I pull the full details on any of these work orders?"
