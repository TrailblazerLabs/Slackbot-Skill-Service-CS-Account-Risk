# Slackbot Skill - Service/CS Account Risk 

## Overview
A Slackbot skill for support/CS teams: give it a customer name and it assembles a risk-flagged brief — ticket history, Slack mentions, CRM account health, and recent email tone — so a manager can see how serious a situation is without digging through four different tools.

## The Problem It Solves
When an account starts looking risky, the information a manager needs to decide whether to step in is scattered: open tickets live in one tool, account health and renewal dates in another, relevant context is buried in Slack threads, and tone shifts show up only in email history nobody has time to reread. Pulling all of that together by hand takes 20–30 minutes and usually happens after something has already gone wrong. This skill compresses that into one on-demand brief with an explicit, reasoned risk level, so escalations get caught earlier and managers get consistent, comparable summaries instead of whatever the requester remembered to check.

## See it in Action

## Quick Start Guide

### Prerequisites
A Slack workspace with Slackbot AI / Skills enabled
At least one ticketing connector enabled (e.g., Zendesk, Jira Service Management, Freshdesk, Intercom, Linear)
A CRM connector enabled if you want account-health data (e.g., Salesforce, HubSpot, Attio) — optional, see setup notes for how to disable this section
An email connector enabled if you want tone analysis (Gmail or Outlook) — optional, see setup notes for how to disable this section
Requester must have Slack search access to the channels you want covered


### Post-Installation Steps
1. Open the skill definition and replace the bracketed placeholders (ticketing system, CRM, email system, priority field names, SLA thresholds) with your team's actual tools and values.
2. If you don't use a CRM or don't want email tone analysis, remove the corresponding steps and output sections rather than leaving them in — an unfilled placeholder will cause the skill to guess or stall.
3. Confirm your org's real SLA thresholds with your support team before filling them in; the defaults in the template are placeholders, not recommendations.
4. Run a test brief on a known account (ideally one with some ticket history) and check that ticket, Slack, and email data are all being pulled correctly before rolling it out to the team.
5. Decide who should be able to invoke this skill — customer data is sensitive, so scope it to the support/CS channels or user groups that should have access, not the whole workspace.
6. Re-run the test after any connector reconnection or scope change, since a broken connector can fail silently depending on how your workspace has it configured.

## About the Creator
Built by @aliwaguespack (https://github.com/aliwaguespack) as part of the Trailblazer Labs Builder in Residence Cohort.
