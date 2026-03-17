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
- **Task** = maintenance (API changes impacting existing app implementation). **Feature** = new capability (not yet in app); **Feature tickets are created only for tier 0 or 1** — never for tier 2.

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

### 4. Decide issue type: Task vs Feature

Based on **API changelog** and **make-custom-app** review of app code:

| Outcome | issueType | When |
|--------|------------|------|
| **Maintenance** | **Task** | API changes **impact existing** implementation: our app already uses the affected endpoints, modules, RPCs, or functions; work is to update, fix, or adapt. |
| **New capability** | **Feature** | Changelog introduces something **new** that is **not covered** in our app (new endpoints, new behavior, new options). **Feature tickets are created only when app tier is 0 or 1.** For tier 2, do not create a Feature for new capabilities. |

- **Rule:** New-capability (Feature) tickets are created **only** for **tier 0 or 1**. For tier 2, create only Task (maintenance) if there is impact on existing code; do not create a Feature.
- Use all inputs (changelog content, current app modules/RPCs/functions, endpoints in use, **app tier**) to classify. One run can result in a Task, a Feature (tier 0/1 only), or both, or no ticket if no impact.

### 5. JIRA: check duplicates, then create or update

- Per [JIRA](.cursor/skills/sdk-api-checkup/JIRA.md): **always check for an existing [API Update] ticket with the same component** (and same issue type if possible) before creating.
- **If existing ticket:** Add a new section to the ticket description: **SDK API Check up — Code analysis** (summary of impact, scope of modules/RPCs/functions affected).
- **If no existing ticket and impact exists:** Create a new ticket per [JIRA](.cursor/skills/sdk-api-checkup/JIRA.md): **issueType** = Task (maintenance) or Feature (new capability) from step 4; Title, Description (API Change Published Date, API Due Date, API Change Description, Scope, Code analysis summary), Team from Airtable, Component = app label, Assignee empty, Due date from API Due Date (tier rule when empty).

### 6. Update Airtable

- In **Monitored apps**, set **Last API Changelog Check Date** to the current date for that app (after processing, whether or not a ticket was created).

---

## Structured outcome

Produce a clear result every run:

| Result | Meaning |
|--------|--------|
| **No new changelog entries** | No ticket. Optionally refresh Last API Changelog Check Date in Airtable. |
| **New entries, no integration impact** | No ticket. Update Last API Changelog Check Date in Airtable. |
| **Impact — existing ticket** | Updated IEN-XXXX with section "SDK API Check up — Code analysis". Airtable updated. |
| **Impact — new ticket (Task)** | Created IEN-XXXX as **Task** (maintenance: existing app code affected). Per JIRA.md. Airtable updated. |
| **Impact — new ticket (Feature)** | Created IEN-XXXX as **Feature** (new capability not in app). **Only for tier 0 or 1.** Per JIRA.md. Airtable updated. |

Always state which of the above applied and, if a ticket was created or updated, its key and issue type (e.g. IEN-XXXX Task, IEN-YYYY Feature).

---

## Context to load

| When | Load |
|------|------|
| **Slack input** | [Slack](.cursor/skills/sdk-api-checkup/Slack.md) — channel, message format, fields to extract. |
| **Airtable data** | [Airtable](.cursor/skills/sdk-api-checkup/Airtable.md) — base, Resource Tracker (team), Monitored apps (changelog, tier, last check). |
| **JIRA ticket** | [JIRA](.cursor/skills/sdk-api-checkup/JIRA.md) — duplicate check, title, description sections, Team, Component, Due date. |
| **Integration impact** | make-custom-app skill — download app code, compare with changelog/API docs. |
