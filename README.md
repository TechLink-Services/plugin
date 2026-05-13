# TechLink Claude Plugin

Extends Claude Code with skills and connectors tailored for TechLink Services — giving every team member access to the SIMPL portal, company data, and operational workflows without switching between tools.

## Skills

### Portal resources
Look up and work with any resource in SIMPL — work orders, clients, projects, technicians, invoices, payments, and scheduling.

- "Find work order #4821."
- "Show me all open jobs for Acme Corp."
- "What's the outstanding balance for this client?"
- "Which technicians are scheduled tomorrow?"

### Query the database
Run aggregate analysis and bulk reports across company data. No SQL knowledge needed — just ask in plain language.

- "How many open work orders do we have right now?"
- "Show me all unpaid invoices older than 30 days."
- "Which technicians have the lowest on-time rate this month?"

### Route jobs
Surface unassigned or unscheduled work orders for a client, grouped by location. From there you can run auto-dispatch, find nearby installers, build routes, or generate route sheets.

- "Show me unscheduled jobs for Acme Corp."
- "What jobs haven't been assigned for this client?"
- "Run auto-dispatch on these open work orders."

### Mesmerize confirmations
Generate a ready-to-send site confirmation email for a Mesmerize work order — pulls the install date, site address, and equipment tracking info from SIMPL automatically.

- "Generate a confirmation email for WO #4821."
- "Send the Mesmerize confirmation for this work order."

### Schema explorer
Understand how the company database is structured — what tables exist, what fields they contain, and how they relate to each other.

- "What tables are in the database?"
- "What fields are on the work order table?"
- "What data do we track about clients?"

## Installation

### Via marketplace (recommended)

Add the TechLink marketplace to Claude Code once, then install the plugin:

```shell
/plugin marketplace add TechLink-Services/claude-plugin
/plugin install techlink-claude-plugin@techlink-claude-plugin
```

Future updates install automatically at startup — no reinstall needed.

### Via .plugin file

If you received a `claude-plugin.plugin` file directly, install it from within Claude Code:

```shell
/plugin install --file /path/to/claude-plugin.plugin
```

## Connector setup

This plugin requires two internal MCP servers (`dev-portal` and `dev-db`) to be configured in Claude Code before the skills will work. Connector setup is handled by the development team as part of onboarding — reach out to them to get your machine configured.
