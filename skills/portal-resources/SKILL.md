---
name: portal-resources
description: >
  Use this skill when the user wants to view, look up, or work with resources in the SIMPL
  portal. Trigger phrases include: "check work orders", "look up a client", "find a job",
  "show me invoices for", "what's the status of", "find technician assignments", "pull up
  the project for", "show me open jobs", "check scheduling", "find a work order", "look up
  a payment", "what work orders are assigned to", "show me what's scheduled", "check the
  portal", "find in SIMPL", or any request to retrieve, review, or summarize information
  from the SIMPL platform. Applies to work orders, clients, projects, technicians,
  invoices, payments, and scheduling.
metadata:
  version: "0.1.0"
---

When this skill triggers, use the dev-portal connector to retrieve and present information from the SIMPL portal in a clear, role-appropriate format.

## What SIMPL manages

SIMPL is TechLink's all-in-one operations platform. It tracks:

- **Work orders** — individual jobs with status, technician assignments, site info, scheduling, and notes
- **Projects** — collections of related work orders, often tied to a single client engagement
- **Clients** — companies or individuals with contact info, billing accounts, and job history
- **Technicians** — field staff with availability, ratings, insurance status, and scheduling
- **Invoices & payments** — billing records, outstanding balances, payment history, and disputes
- **Scheduling** — which jobs are assigned, when they're booked, and what's unscheduled

## Core behavior

1. Identify what the user is looking for (a specific record, a list, a status check, or a summary).
2. Use the appropriate dev-portal tool to retrieve the relevant data.
3. Present results in a format suited to the request — a single record gets a structured summary; a list gets a scannable table; a status check gets a one-line answer with key details.
4. Offer relevant follow-ups based on what was retrieved (e.g., "Want me to pull the invoice history for this client?" or "Should I check which technician is assigned?").

## Role-specific framing

Tailor the presentation to the likely audience:
- **Operations**: Focus on job status, scheduling gaps, technician assignments, and urgency.
- **Accounting**: Focus on invoice amounts, payment status, outstanding balances, and billing contacts.
- **Sales**: Focus on client history, project volume, and relationship signals.
- **Executives**: Lead with summary metrics and flag anything requiring attention.
- **Recruiting**: Focus on technician profiles, ratings, and credential status.

## Limits

This skill will change data in the SIMPL portal system, any endpoint the begins with 'get' is read-only, otherwise it is making a change to the data
