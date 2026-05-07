---
name: query-db
description: >
  Use this skill when the user wants to pull data or get answers from the company database.
  Trigger phrases include: "query the database", "pull data on", "show me records for",
  "how many", "list all", "look up", "find records", "give me a report on", "what does the
  data show for", "run a query", or any request for numbers, counts, lists, or summaries
  that would require database access. Suitable for all team members regardless of technical
  background — translate natural language into structured queries automatically.
metadata:
  version: "0.1.0"
---

When this skill triggers, help the user retrieve data from the company database via the dev-db connector without requiring them to know SQL or technical database concepts.

## Core behavior

1. Clarify the request if needed — ask one focused question if the intent is ambiguous (e.g., which time period, which client, which status).
2. Translate the user's plain-language request into the appropriate query using the dev-db connector tools.
3. Present results in a clean, readable format — use a table for lists, plain numbers for counts, and a brief summary sentence at the top.
4. Offer a natural follow-up (e.g., "Want me to filter this by date range or export it?").

## Audience

Users range from non-technical (accounting, recruiting, marketing) to technical (developers, operations). Always communicate results in plain language. Never expose raw SQL or technical error messages to non-technical users — translate errors into plain descriptions of what went wrong and what to try instead.

## Useful patterns by department

See `references/query-examples.md` for ready-to-use example queries organized by department (operations, accounting, sales, recruiting, marketing, executives).

## Query guidelines

The `query` tool's description contains the full, authoritative guidelines for this database — approved table names, column exclusions, date handling, polymorphic joins, and more. **Always read the `query` tool description before writing any SQL.** Do not guess table or column names; call `describe_table` for every table you plan to use before writing a query.

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
