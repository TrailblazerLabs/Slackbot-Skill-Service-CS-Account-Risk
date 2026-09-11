# Slackbot Skill - Service/CS Account Risk 

## Overview
A Slackbot skill for support/CS teams: give it a customer name and it assembles a risk-flagged brief — ticket history, Slack mentions, CRM account health, and recent email tone — so a manager can see how serious a situation is without digging through four different tools.

## The Problem It Solves
When an account starts looking risky, the information a manager needs to decide whether to step in is scattered: open tickets live in one tool, account health and renewal dates in another, relevant context is buried in Slack threads, and tone shifts show up only in email history nobody has time to reread. Pulling all of that together by hand takes a lot of time and usually happens after something has already gone wrong. This skill compresses that into one on-demand brief with an explicit, reasoned risk level, so escalations get caught earlier and managers get consistent, comparable summaries instead of whatever the requester remembered to check.

## See it in Action

## Quick Start Guide

### Prerequisites
A Slack workspace with Slackbot AI / Skills enabled
At least one ticketing connector enabled (e.g., Zendesk, Jira Service Management, Freshdesk, Intercom, Linear)
A CRM connector enabled if you want account-health data (e.g., Salesforce, HubSpot, Attio) — optional, see setup notes for how to disable this section
An email connector enabled if you want tone analysis (Gmail or Outlook) — optional, see setup notes for how to disable this section
Requester must have Slack search access to the channels you want covered


### Configuring the Skill File for Your Tool Stack

The skill definition (the `.md` file you paste into Slackbot's Skill builder) is written generically with bracketed placeholders instead of a specific ticketing tool, CRM, or email provider. Before using it, open the file and do a find-and-replace for each placeholder below.

| Placeholder in the file | Replace with | Example |
|---|---|---|
| `[TICKETING_SYSTEM]` | Your ticketing/support tool | Zendesk, Jira Service Management, Freshdesk, Intercom, Linear |
| `[CRM_SYSTEM]` | Your CRM / account-health source | Salesforce, HubSpot, Attio, internal dashboard |
| `[EMAIL_SYSTEM]` | Your email connector | Gmail, Outlook |
| `[TICKET_ID_FIELD]` | What tickets are called/numbered in your tool | "Zendesk ticket #", "Jira issue key" |
| `[PRIORITY_FIELD]` | Your tool's priority/severity naming | P1–P3, Urgent/High/Normal/Low, Sev1–Sev3 |
| `[SLA_P1]` / `[SLA_P2]` / `[SLA_P3]` | Your org's actual SLA thresholds | 4 hours / 1 business day / 3 business days |

**How to do it:**
1. Open the skill `.md` file in any text editor (or directly in the Skill builder once pasted in).
2. Search for `[` to jump to every placeholder — each one appears in more than one place in the file (instructions, risk rubric, and output format all reference the same placeholders), so a single find-and-replace per placeholder is faster than editing line by line.
3. Replace each placeholder consistently — use the exact same name everywhere (e.g., always "Zendesk," not "Zendesk" in one spot and "our ticketing tool" in another), since the wording is what tells Slackbot which connector to call.
4. Save, then paste the full updated file into the Skill builder.

**If you don't have a CRM connector:** delete the CRM-related instruction step and the "Account snapshot" section in the output format, and remove the renewal-date risk triggers from the rubric — they depend on CRM data and will cause the skill to stall or guess if left in unfilled.

**If you don't have an email connector for this skill:** delete the email-tone instruction step and the "Email tone" output section, and remove the "negative tone shift" risk trigger from the rubric.

**If you use more than one ticketing system** (e.g., a support tool plus an internal Jira for engineering escalations): duplicate the ticket-pulling step for each system, and note in the output which system each ticket came from.

**How Slack Connects to Jira, Zendesk, and Other Systems**

The connectors mentioned above (ticketing tool, CRM, email) aren't things you configure inside the skill file itself — they're separate integrations set up at the Slack workspace level, using Slack's Model Context Protocol (MCP) support. Here's how that actually works:

The mechanism: MCP is an open-source framework for connecting AI applications with other software systems, and Slackbot can be set up as an MCP client so it can take actions in other apps. When you install an app that includes an MCP server, anyone with access to Slackbot can interact with that service just by starting a conversation — Slackbot gains access to a set of tools configured by the app's developer to take actions in that app from Slack. Those tools are typically split into read tools (so Slackbot can search and return information) and write tools (so it can create or update content) — which ones are available depends entirely on what the app's developer built.

Then each user connects their own account: as with other Slack apps, individual members need to connect their own account before they can use the integration, and once connected, Slackbot has access to all of the tools built into that app's MCP server — meaning the skill will only be able to pull Jira/Zendesk/CRM data for people who've done this, not the whole workspace automatically.

What this means for rolling the skill out: before anyone can use this skill, an admin needs to install the relevant apps (ticketing, CRM, email) from the Slack Marketplace, and each person who'll run the skill needs to individually authorize their own account for each one. It's worth doing this and confirming the connection works before editing the skill file's placeholders, so you're not troubleshooting two things at once.

### Post-Installation Steps
1. Open the skill definition and replace the bracketed placeholders (ticketing system, CRM, email system, priority field names, SLA thresholds) with your team's actual tools and values — see **Configuring the Skill File for Your Tool Stack** above for the full walkthrough.
2. If you don't use a CRM or don't want email tone analysis, remove the corresponding steps and output sections rather than leaving them in — an unfilled placeholder will cause the skill to guess or stall.
3. Confirm your org's real SLA thresholds with your support team before filling them in; the defaults in the template are placeholders, not recommendations.
4. Run a test brief on a known account (ideally one with some ticket history) and check that ticket, Slack, and email data are all being pulled correctly before rolling it out to the team.
5. Decide who should be able to invoke this skill — customer data is sensitive, so scope it to the support/CS channels or user groups that should have access, not the whole workspace.
6. Re-run the test after any connector reconnection or scope change, since a broken connector can fail silently depending on how your workspace has it configured.

## About the Creator
Built by @aliwaguespack (https://github.com/aliwaguespack) as part of the Trailblazer Labs Builder in Residence Cohort.
