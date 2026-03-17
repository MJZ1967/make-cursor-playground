---
name: sdk-api-checkup
description: Runs the full API Check Up process: parse Slack input, fetch Airtable data, check changelog, analyze integration impact with make-custom-app, and produce a structured outcome (JIRA ticket and/or Airtable update). Use when processing a Slack message from #apps-api-checkups, when the user runs a manual API check up, or when coordinating Slack + Airtable + JIRA for an app (component).
---

# API Check Up — Process

This skill defines **how to run the whole process** and **what structured outcome** to produce. Input definitions live in [Slack](.cursor/skills/sdk-api-checkup/Slack.md), [Airtable](.cursor/skills/sdk-api-checkup/Airtable.md), and [JIRA](.cursor/skills/sdk-api-checkup/JIRA.md). Load those when you need channel format, fields, or ticket structure.

**Vocabulary (aligned across skills):**

- **Component** = **app** = **app label** — same identifier in Slack message, Airtable (App Name), and JIRA (component).
- **Changelog URL** — from Slack message or Airtable (Monitored apps).
- **Tier** — T0 / T1 / T2 from Slack or Airtable (**app tier**).
- **Team** — Athena | Atlantis | Plato | Hermes from Airtable (Resource Tracker or Monitored apps).

---

## Triggers

| Trigger | When |
|--------|------|
| **Slack** | New message in **#apps-api-checkups** or user pastes a message. |
| **Manual** | User asks to run the API Check Up or to process a specific app (component). |

---

## Process (steps)

### 1. Get input

- **From Slack (or user):** Parse per [Slack](.cursor/skills/sdk-api-checkup/Slack.md). Extract **app name** (component), **Tier**, **Changelog URL**.
- **From Airtable:** Per [Airtable](.cursor/skills/sdk-api-checkup/Airtable.md), look up the app in **Monitored apps** by App Name. Get **Last API Changelog Check Date**, **app tier**, **team**, and Changelog/API docs URL if not from Slack.

### 2. Check changelog for new entries

- Fetch or parse the **Changelog URL** (or API docs URL from Airtable).
- Compare with **Last API Changelog Check Date**.
- **If no new entries** → go to **Structured outcome** with result: no ticket; optionally update Last API Changelog Check Date in Airtable.

### 3. Analyze integration impact (if new entries exist)

- Use **make-custom-app** skill: download app code (component = app slug/version), inspect endpoints, auth, payloads, behavior.
- Compare app usage to changelog/deprecation content.
- Decide: **Impact** (Y/N). If Y, list **changes needed** (e.g. endpoints, auth, scopes, TLS).

### 4. JIRA: check duplicates, then create or update

- Per [JIRA](.cursor/skills/sdk-api-checkup/JIRA.md): **always check for an existing [API Update] ticket with the same component** before creating.
- **If existing ticket:** Add a new section to the ticket description: **SDK API Check up — Code analysis** (summary of impact, scope of modules/RPCs/functions affected).
- **If no existing ticket and impact exists:** Create a new ticket per [JIRA](.cursor/skills/sdk-api-checkup/JIRA.md): Title, Description (API Change Published Date, API Due Date, API Change Description, Scope), Team from Airtable, Component = app label, Assignee empty, Due date from API Due Date (tier rule when empty).

### 5. Update Airtable

- In **Monitored apps**, set **Last API Changelog Check Date** to the current date for that app (after processing, whether or not a ticket was created).

---

## Structured outcome

Produce a clear result every run:

| Result | Meaning |
|--------|--------|
| **No new changelog entries** | No ticket. Optionally refresh Last API Changelog Check Date in Airtable. |
| **New entries, no integration impact** | No ticket. Update Last API Changelog Check Date in Airtable. |
| **Impact — existing ticket** | Updated IEN-XXXX with section "SDK API Check up — Code analysis". Airtable updated. |
| **Impact — new ticket** | Created IEN-XXXX per JIRA.md (title, description, Scope, Team, Component, Due date). Airtable updated. |

Always state which of the above applied and, if a ticket was created or updated, its key (e.g. IEN-XXXX).

---

## Context to load

| When | Load |
|------|------|
| **Slack input** | [Slack](.cursor/skills/sdk-api-checkup/Slack.md) — channel, message format, fields to extract. |
| **Airtable data** | [Airtable](.cursor/skills/sdk-api-checkup/Airtable.md) — base, Resource Tracker (team), Monitored apps (changelog, tier, last check). |
| **JIRA ticket** | [JIRA](.cursor/skills/sdk-api-checkup/JIRA.md) — duplicate check, title, description sections, Team, Component, Due date. |
| **Integration impact** | make-custom-app skill — download app code, compare with changelog/API docs. |
