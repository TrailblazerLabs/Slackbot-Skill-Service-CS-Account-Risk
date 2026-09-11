# Skill: Customer Escalation Brief

## ⚙️ Setup — edit this section for your tool stack

This skill is written generically. Before using it, replace the bracketed
placeholders below with your actual tools, then use Find & Replace to swap
every instance of that placeholder throughout the rest of the file.

| Placeholder | Replace with | Example |
|---|---|---|
| `[TICKETING_SYSTEM]` | Your ticketing/support tool | Zendesk, Jira Service Management, Freshdesk, Intercom, Linear |
| `[CRM_SYSTEM]` | Your CRM / account-health source | Salesforce, HubSpot, Attio, internal dashboard |
| `[EMAIL_SYSTEM]` | Your email connector | Gmail, Outlook |
| `[TICKET_ID_FIELD]` | What tickets are called/numbered as in your tool | "Zendesk ticket #", "Jira issue key" |
| `[PRIORITY_FIELD]` | Your tool's priority/severity naming | P1–P3, Urgent/High/Normal/Low, Sev1–Sev3 |
| `[SLA_P1]` / `[SLA_P2]` / `[SLA_P3]` | Your org's actual SLA thresholds | 4 hours / 1 business day / 3 business days |

**No CRM?** Delete step 2 and the "Account snapshot" output section, and
remove the renewal-date risk triggers from the rubric — they depend on
CRM data.

**No email connector enabled for this skill?** Delete step 5 and the
"Email tone" output section, and remove the "negative tone shift" risk
trigger from the rubric.

**Multiple ticketing systems (e.g., a support tool + an internal Jira for
engineering escalations)?** Duplicate step 3 for each system, and note in
the output which system each ticket came from.

Everything below this line uses the placeholders — search for `[` to find
every spot that references a connector-specific detail.

---

## Purpose
Given a customer name, assemble a risk-flagged summary for a manager by pulling
ticket history, Slack mentions, [CRM_SYSTEM] account health, and recent email
tone into one structured brief. Built for support/CS teams who need a fast,
consistent read on "how bad is this, really" before a manager steps in.

## Required connectors
- [CRM_SYSTEM] (account, opportunity, and case/health data)
- [TICKETING_SYSTEM] (ticket history)
- Slack search (across channels the requester has access to)
- [EMAIL_SYSTEM] (for recent thread tone)
- Slackbot surfaces (built-in — no separate connector needed, but requires
  a plan where surfaces are available)

If any connector is missing or returns an error, do not stop — see
**Fallback behavior** below.

## Trigger
Run when a user provides a customer or account name and asks for an
escalation brief, risk read, or "what's going on with [account]"-style
request. If no customer name is given, ask for one before proceeding —
do not guess or default to the most recently mentioned account.

## Instructions (step-by-step)

1. **Resolve the account.** Search [CRM_SYSTEM] for the account matching the
   given name. If more than one account plausibly matches (e.g. similar
   names, multiple regional subsidiaries), list the candidates and ask
   the user to confirm which one before continuing.

2. **Pull [CRM_SYSTEM] account health.**
   - Account tier / ARR / renewal date
   - Open opportunities and their stage
   - Any existing case escalations or health scores if the org tracks them
   - Note the data pull date/time so the brief shows its own freshness

3. **Pull ticket history from [TICKETING_SYSTEM].**
   - All tickets ([TICKET_ID_FIELD]) from this account in the last 90 days
   - For each: status, [PRIORITY_FIELD], age (days open), and one-line summary
   - Flag any ticket that has been reopened more than once
   - Flag any ticket open longer than the org's stated SLA for its priority
     — use [SLA_P1] / [SLA_P2] / [SLA_P3] as thresholds. If these haven't
     been filled in above, ask the user for their org's SLA thresholds
     rather than guessing.

4. **Search Slack for mentions.**
   - Search the customer/account name across channels the requester can
     access, last 30 days
   - Pull the surrounding thread context, not just the matching message
   - Distinguish between internal discussion about the customer and any
     messages in a shared/external channel with the customer directly
   - Note the tone of internal mentions (e.g., "team flagged frustration,"
     "routine status update") without inventing sentiment that isn't there

5. **Check recent email tone via [EMAIL_SYSTEM].**
   - Pull the last 5–10 email threads involving this customer
   - Summarize tone shifts in plain language (e.g., "tone has cooled over
     the last two threads — shorter replies, no longer cc'ing their team")
   - Do not quote email content verbatim in the brief; paraphrase only

6. **Apply the risk-flagging rubric** (see below) to assign an overall
   risk level, and briefly justify it with the specific signals that drove
   it — do not just state a color/label without the reasoning.

7. **Build an interactive surface** presenting the brief — see **Surface
   layout** below. Don't just paste the text brief into a surface; use the
   sortable-list and visualization capabilities so the manager can filter
   and scan rather than read a wall of text.

8. **Share the surface** in the channel or DM where the request was made,
   and also post a short plain-text summary alongside it (risk level +
   one-line reason) so the headline is visible without opening the surface.

9. **Also produce the text brief** in the output format below, as a
   fallback for anyone who can't access surfaces or wants a plain copy for
   a doc/email.

## Risk-flagging rubric
Score each signal, then take the highest triggered level as the overall
risk (don't average them down — one severe signal should surface, not get
diluted):

- **🔴 High risk** — any of: SLA breach on an open high-priority ticket
  (e.g. [SLA_P1]/[SLA_P2] tier), ticket reopened 2+ times, renewal date
  within 60 days with an unresolved escalation, clear negative tone shift
  in email, or explicit churn/cancel language found in tickets, Slack, or
  email.
- **🟡 Medium risk** — any of: SLA breach on a lower-priority ticket
  (e.g. [SLA_P3] tier), multiple open tickets with no high-priority ones,
  internal Slack chatter expressing concern without an explicit threat, or
  a renewal within 90 days with no other red flags.
- **🟢 Low risk** — none of the above; routine ticket volume, stable or
  positive tone, no near-term renewal risk.

State which specific signals triggered the level. If signals conflict
(e.g., positive email tone but an SLA breach), say so explicitly rather
than smoothing it over.

## Surface layout
When building the surface, ask for these components explicitly rather than
letting Slackbot default to a plain text dump:

- **Header** — account name, risk level (color-coded), data pull date/time
- **Risk summary panel** — the 2-3 sentence "why this risk level" reasoning,
  displayed prominently, not buried under the data
- **Ticket table** — sortable by priority, status, and age, from
  [TICKETING_SYSTEM]; SLA breaches and reopens visually flagged (e.g. a
  warning icon or highlighted row), not just noted in text
- **Account health panel** — tier/ARR/renewal date from [CRM_SYSTEM], with
  renewal date visually called out if it falls within 90 days
- **Slack & email activity feed** — chronological, most recent first,
  labeled by source
- **Insights section** — Slackbot's own "additional insights, key themes,
  or recommended action items" panel (built into surfaces); make sure the
  suggested next step from step 6 lands here, not just in the text brief

Note: surfaces are static snapshots — they won't auto-refresh if ticket or
CRM data changes after creation. Mention this in the surface itself (e.g.
"Data as of [date/time] — re-run this skill for an updated view") so
managers don't mistake it for a live dashboard.

## Output format

```
# Escalation Brief: [Account Name]
Data pulled: [date/time] | Risk level: [🔴/🟡/🟢] [High/Medium/Low]

## Why this risk level
[2-3 sentences naming the specific signals that drove the rating]

## Account snapshot ([CRM_SYSTEM])
- Tier / ARR: ...
- Renewal date: ...
- Open opportunities: ...

## Ticket history — [TICKETING_SYSTEM] (last 90 days)
- [ticket ID] — [priority] — [status] — [age] — [one-line summary] [⚠ if SLA breach or reopened]
  (repeat, most severe/oldest first)

## Slack activity (last 30 days)
- [channel] — [date] — [one-line context] [note if internal-only vs. customer-facing channel]
  (omit this section entirely if nothing relevant was found — don't say "none found")

## Email tone
[2-3 sentence summary of tone and any shift, no verbatim quotes]

## Suggested next step for the manager
[One specific, actionable suggestion — e.g., "recommend a proactive call
before the renewal date" — not a generic "monitor the situation"]
```

## Fallback behavior
- If [CRM_SYSTEM] is unavailable: proceed with ticket/Slack/email data, and
  state clearly at the top of the brief that account health data is
  missing and the risk level should be treated as provisional.
- If no tickets are found: state that explicitly rather than omitting the
  section silently — "no tickets found in the last 90 days" is itself a
  useful signal.
- If a connector returns an auth/permission error: name which connector
  failed so the user can reconnect it, rather than silently skipping it.
- Never fabricate data to fill a gap. An incomplete brief with clearly
  marked gaps is more useful than a complete-looking one with invented
  numbers.

## Edge cases to handle
- Customer name matches no [CRM_SYSTEM] account: say so and ask if the
  account might be under a different name (e.g., parent company vs.
  subsidiary).
- [TICKETING_SYSTEM] and [CRM_SYSTEM] disagree on account status (e.g.,
  account marked "closed won" but tickets are still open): flag the
  discrepancy, don't silently pick one source.
- Very high-volume accounts (50+ tickets in 90 days): summarize by
  category/theme instead of listing every ticket, but still surface
  individual SLA breaches or reopens by name.

## Example use
**User:** "Give me an escalation brief on Meridian Logistics"
**Skill:** resolves the [CRM_SYSTEM] account, pulls the last 90 days of
tickets from [TICKETING_SYSTEM], searches Slack for "Meridian" across
accessible channels, checks recent email threads with Meridian contacts
via [EMAIL_SYSTEM], applies the risk rubric, builds and shares an
interactive surface with a sortable ticket table and risk summary, and
also returns the formatted text brief above as a fallback.
