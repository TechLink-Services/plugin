---
name: query-db
description: >
  Use this skill only when the user explicitly needs aggregate analysis, bulk reporting, or
  cross-table queries that the SIMPL portal API cannot satisfy. Trigger phrases include:
  "query the database", "run a query", "give me a report on", "what does the data show for",
  "how many total", "aggregate", "sum up", "month-over-month", "year-over-year", or any
  request requiring custom SQL or analysis across multiple data sources. Do NOT use this
  skill for simple record lookups, status checks, or listing individual resources — use the
  portal-resources skill for those instead. Suitable for all team members regardless of
  technical background — translate natural language into structured queries automatically.
metadata:
  version: "0.1.0"
---

When this skill triggers, help the user retrieve data from the company database via the dev-db connector without requiring them to know SQL or technical database concepts.

## MANDATORY WORKFLOW — follow every step, no exceptions

**Step 1 — Check for a saved query tool first.**
The connector exposes pre-built tools for common requests (`user_account_list`, `workorder_browse_by_client`, `client_invoice_list`, etc.). If one matches the request, use it — do not write raw SQL. Check the domain reference below to pass the right parameter values.

**Step 2 — If writing raw SQL, describe every table first.**
Call `describe_table` for each table you plan to use. Do this before writing a single line of SQL. This database has non-standard column names — `users` has no `id`, `name`, or `role` column; `workorder` has no `status` column. You will get it wrong if you guess.

**Step 3 — Show the columns you found.**
After calling `describe_table`, tell the user (briefly): "Found columns: …" before writing the query. This step is visible and skipping it is a failure.

**Step 4 — Write and run the query using only confirmed column names.**

**Step 5 — Present results clearly.**
Use a table for lists, plain numbers for counts, a brief summary sentence at the top. Offer a relevant follow-up.

## Audience

Users range from non-technical (accounting, recruiting, marketing) to technical (developers, operations). Always communicate results in plain language. Never expose raw SQL or technical error messages to non-technical users — translate errors into plain descriptions of what went wrong and what to try instead.

## Query guidelines

This database has inconsistent naming conventions — some tables are singular (`workorder`, `client`, `task`), others plural (`tech_invoices`, `users`). Never guess a table name. The `query` tool's description contains the full approved table list and all SQL guidelines (date handling, polymorphic joins, active record filtering, etc.).

## Domain reference

Use these values directly — do not query the database to look them up.

### User roles (`users.roles` — bitmask, use `& value` not `= value`)

| Value | Role |
|---|---|
| 1 | Admin |
| 2 | Employee |
| 4 | Client |
| 8 | Installer (field technician) |
| 16 | Applicant |
| 32 | Approver |
| 64 | Sales |
| 128 | Accountant |
| 256 | Director |
| 512 | Customer |

Example: to list directors, call `user_account_list` with `role_flag=256`.

### Work order statcodes (`workorder.statcode`)

| Value | Status |
|---|---|
| 10 | Created |
| 20 | Open |
| 22 | Dispatched |
| 40 | Assigned |
| 50 | Accepted |
| 60 | Scheduled |
| 70 | On Site |
| 75 | Incomplete |
| 78 | Off Site / Return Needed |
| 80 | In Review |
| 90 | Approved |
| 100 | Closed |
| 105 | Cancelled |

### Note types (`note.context`)

| Value | Type |
|---|---|
| 1 | System note (auto-generated) |
| 2 | User note (written by a person) |

### Task types (`task.tasktype_id`)

| Value | Type |
|---|---|
| 1 | RegularTask (live tasks — use for billing/finance) |
| 2 | QuoteTask (pending-approval quote tasks) |
| 3 | TaskAtInvoiceTime (frozen invoice snapshot) |

## Data boundaries

- Only query data the dev-db connector exposes. Do not attempt to access tables or fields not surfaced by the connector.
- If a query would return a very large result set (hundreds of rows), proactively offer to summarize or filter rather than dumping everything.
- Never modify data through this skill — reads only. For data changes, direct the user to the SIMPL portal.
