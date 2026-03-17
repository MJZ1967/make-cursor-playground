---
name: sdk-api-checkup
description: Runs the full API Check Up process: parse Slack input, fetch Airtable data, check changelog, analyze integration impact with make-custom-app, and produce a structured outcome (JIRA ticket and/or Airtable update). Use when processing a Slack message from #apps-api-checkups, when the user runs a manual API check up, or when coordinating Slack + Airtable + JIRA for an app (component).
---

# API Check Up — Process

This skill defines **how to run the whole process** and **what structured outcome** to produce. Slack input format, Airtable fields, and JIRA ticket structure are all defined inline below.

**Vocabulary (aligned across the skill):**

- **Component** = **app** = **app label** — same identifier in Slack message, Airtable (App Name), and JIRA (component).
- **Changelog URL** — from Slack message or Airtable (Monitored apps).
- **Tier** — T0 / T1 / T2 / T3 / T4 from Slack or Airtable (**app tier**). We recognize tiers 0, 1, 2, 3, 4.
- **Team** — Athena | Atlantis | Plato | Hermes from Airtable (Resource Tracker or Monitored apps).
- **Task** = maintenance (API changes impacting existing app implementation). **Feature** = new capability (not yet in app); **Feature tickets are created only for tier 0 or 1** — never for tier 2, 3, or 4.

---

## Triggers

| Trigger | When |
|--------|------|
| **Slack** | New message in **#apps-api-checkups** or user pastes a message. |
| **Manual** | User asks to run the API Check Up or to process a specific app (component). |

---

## Rules

- **Always skip messages that already have thread replies.** When reading messages from **#apps-api-checkups**, skip any message that already has one or more replies in its thread (i.e. `reply_count > 0` or a `thread_ts` with existing replies). These messages have already been processed. Only process messages with no thread replies yet.
- **Always skip messages from "Missing API Status".** When reading from **#apps-api-checkups** (or when the user pastes channel history), ignore any message whose sender or display name is **Missing API Status**. Do not parse, process, or run the API Check Up workflow for those messages. Only process messages from other senders (e.g. human users or other bots that request a manual check).
- **Always run code analysis comparison.** When the changelog has new entries (or when creating/updating an [API Update] ticket), **always** download the app code via make-custom-app (e.g. `download-app.js`), inspect endpoints, auth, and payloads, and compare app usage to the changelog/deprecation content. Never create or update a ticket with placeholder Scope (e.g. "To be confirmed") or without a code-based **SDK API Check up — Code analysis** section. The Scope table and Code analysis must reflect the actual app structure (modules, RPCs, functions, connections, webhooks) and impact.
- **Always include "What to do in the code" when there are code changes.** Every ticket or analysis that has integration impact and **requires code changes** must include a **What to do in the code** section with concrete, actionable items: which file(s) or component(s) to change, what to add/remove/verify, and paths or component names. No vague "review" or "consider" only — list implementable steps so a developer can apply the API changes without re-reading the changelog.
- **Skip code analysis outcome points when outcome = no change.** When the code analysis concludes **no code change required** (e.g. app already compliant, changelog is docs-only with no implementation impact, or only verification needed with no file edits), do **not** require a full **What to do in the code** table or list. Instead state briefly in the ticket/analysis: "No code change required" and optionally what was verified (e.g. auth, endpoints). Omit or replace the detailed actionable items; Scope and Code analysis summary may still be included for traceability.
- **Do not create a Task for tier 3 or 4 unless the change is breaking.** A breaking change means: deprecation with end-of-life date, endpoint removal, authentication/auth flow change, or a forced migration that will break existing scenarios if ignored. Non-breaking additions, optional new parameters, or documentation-only changes do not qualify. For tier 0, 1, or 2, create a Task for any integration impact.
- **Always reply in the Slack thread of the source message after processing, and mark the parent message.** When the input came from a Slack message (channel `#apps-api-checkups`), after completing the process: (1) post a thread reply using `slack_send_message` with `thread_ts` = the source message's `message_ts` per the format in step 7; (2) mark the parent message with reactions `:white_check_mark:` (analysis done) and `:ai-make:` — use the Slack MCP reaction tool if available, otherwise note in the thread reply that the message was processed (the Slack MCP currently does not expose an add-reaction tool, so reactions cannot be added programmatically at this time).

---

## Process (steps)

### 1. Get input

- **From Slack (or user):** Apply filters in order before processing: (1) skip if the message is from **Missing API Status**; (2) skip if the message already has thread replies (`reply_count > 0`). Only messages that pass both filters are valid. For valid messages, parse per [Slack input](#slack-input) and extract **app name** (component), **Tier**, **Changelog URL**.
- **From Airtable:** Per [Airtable input](#airtable-input), look up the app in **Monitored apps** by App Name. Get **Last API Changelog Check Date**, **app tier**, **team**, and Changelog/API docs URL if not from Slack.

### 2. Check changelog for new entries

- Fetch or parse the **Changelog URL** (or API docs URL from Airtable).
- Compare with **Last API Changelog Check Date**.
- **If no new entries** → go to **Structured outcome** with result: no ticket; optionally update Last API Changelog Check Date in Airtable.

### 3. Analyze integration impact (if new entries exist)

- **Mandatory:** Always run code analysis comparison. Use **make-custom-app** skill: download app code (component = app slug/version) via the download script, inspect endpoints, auth, payloads, behavior.
- Compare app usage to changelog/deprecation content. Fill the **Scope** table and **SDK API Check up — Code analysis** from the downloaded app (real module/RPC/function/connection/webhook names and impact), not placeholders.
- **What to do in the code:** For every impact (Y) **that requires code changes**, produce a concrete **What to do in the code** section with actionable items (path/component, current behaviour, required change). When the outcome is **no code change required**, skip the detailed table/list; state "No code change required" and optionally what was verified (see Rules: skip code analysis outcome points when = no change).
- Decide: **Impact** (Y/N). If Y, list **changes needed** (e.g. endpoints, auth, scopes, TLS). If Y but **no code change required**, do not add a full "What to do in the code" list — state the no-change conclusion only.

### 4. Decide issue type: Task vs Feature

Based on **API changelog** and **make-custom-app** review of app code:

| Outcome | issueType | When |
|--------|------------|------|
| **Maintenance** | **Task** | API changes **impact existing** implementation: our app already uses the affected endpoints, modules, RPCs, or functions; work is to update, fix, or adapt. **For tier 3 or 4: create Task only when the change is breaking** (deprecation, removal, auth change, forced migration). For tier 0, 1, or 2: create Task for any impact. |
| **New capability** | **Feature** | Changelog introduces something **new** that is **not covered** in our app (new endpoints, new behavior, new options). **Feature tickets are created only when app tier is 0 or 1.** For tier 2, 3, or 4, do not create a Feature for new capabilities. |

- **Rule:** New-capability (Feature) tickets are created **only** for **tier 0 or 1**. For tier 2, 3, or 4, create only Task (maintenance) if there is impact on existing code; do not create a Feature.
- **Rule:** Task (maintenance) tickets for **tier 3 or 4** are created **only when the change is breaking** — i.e. deprecation, endpoint removal, auth change, or forced migration that will break existing scenarios if not acted on. Non-breaking improvements or additions do not warrant a Task for tier 3 or 4.
- Use all inputs (changelog content, current app modules/RPCs/functions, endpoints in use, **app tier**) to classify. One run can result in a Task, a Feature (tier 0/1 only), or both, or no ticket if no impact.

### 5. JIRA: check duplicates, then create or update

- Per [JIRA structure](#jira-structure): **always check for an existing [API Update] ticket with the same component** (and same issue type if possible) before creating.
- **If existing ticket:** Add a new section to the ticket description: **SDK API Check up — Code analysis** and, when code changes are required, **What to do in the code**. When outcome = no change, omit the detailed "What to do in the code" and state "No code change required" (and what was verified) instead.
- **If no existing ticket and impact exists:** Create a new ticket per [JIRA structure](#jira-structure): **issueType** = Task (maintenance) or Feature (new capability) from step 4; Title, Description (API Change Published Date, API Due Date, API Change Description, Scope, Code analysis summary, and **What to do in the code** only when code changes are required; when no change, state "No code change required" instead), Team from Airtable, Component = app label, Assignee empty, Due date from API Due Date (tier rule when empty).

### 6. Update Airtable

- In **Monitored apps**, set **Last API Changelog Check Date** to the current date for that app (after processing, whether or not a ticket was created).

### 7. Reply in Slack thread

After completing the process for each app, post a reply in the thread of the source Slack message using `slack_send_message` with `thread_ts` = source message's `message_ts` and `channel_id` = `C043TH2131Q`.

**Reply format depends on the outcome:**

| Outcome | Reply content |
|---------|--------------|
| **JIRA ticket created or updated** | `:jira: *IEN-XXXX created/updated — [App label] (T[tier])*` + one-line description of the change and issue type (Task/Feature). Example: `:jira: *IEN-12345 created — airtable (T0)* — Task: deprecation of offset-based pagination affects listRecords module.` |
| **No new entries / No integration impact / No code change** | `:white_check_mark: *API Check Up — [app label] (T[tier])*` + bullet list of: changelog period checked, key findings (2–4 bullets), result (`No code change required` / `No new entries`). See example reply sent for whatsapp-business-cloud, active-directory, pinterest in this conversation. |

**Rules for the thread reply and parent message marking:**
- One reply per app processed (one `slack_send_message` call per source message).
- Always use `thread_ts` — never post to the channel root.
- Keep the reply concise: max ~10 lines.
- Always include the check date and the last-check date for traceability.
- **After posting the reply, add reactions to the parent message:** `:white_check_mark:` (analysis complete) and `:ai-make:` (processed by AI). Use the Slack MCP add-reaction tool if available. If the MCP does not support reactions, skip silently — do not block or fail the process.

---

## Structured outcome

Produce a clear result every run:

| Result | Meaning |
|--------|--------|
| **No new changelog entries** | No ticket. Optionally refresh Last API Changelog Check Date in Airtable. |
| **New entries, no integration impact** | No ticket. Update Last API Changelog Check Date in Airtable. |
| **Impact — existing ticket** | Updated IEN-XXXX with section "API Check up — Code analysis". Airtable updated. |
| **Impact — new ticket (Task)** | Created IEN-XXXX as **Task** (maintenance: existing app code affected). **For tier 3 or 4: only when the change is breaking** (deprecation, removal, auth change, or forced migration). For tier 0, 1, or 2: create Task for any impact. Per JIRA structure. Airtable updated. |
| **Impact — new ticket (Feature)** | Created IEN-XXXX as **Feature** (new capability not in app). **Only for tier 0 or 1** (never 2, 3, or 4). Per JIRA structure. Airtable updated. |

Always state which of the above applied and, if a ticket was created or updated, its key and issue type (e.g. IEN-XXXX Task, IEN-YYYY Feature).

---

## Slack input

Simple overview of the Slack source used by the API Check Up process.

### Channel

| Item | Value |
|------|--------|
| **Channel** | `#apps-api-checkups` Channel = 'C043TH2131Q'|
| **Purpose** | One message per app = one component that needs an API/changelog check. |

### Message = one app (component)

- **One message** in the channel = **one app** = **one component**.
- The app is represented as a **block of code** in the message: `[app]` (e.g. code block or similar delimiter in the actual message).
- Use the content of that block as the **app identifier** (e.g. `airtable`, `openai-gpt-3`).

### What to look for in each message

| Field | What to extract | Example |
|-------|------------------|--------|
| **App name** | From the message text or the `[app]` block | `airtable` |
| **Tier** | T0 / T1 / T2 / T3 / T4 (in parentheses in the text). We recognize tiers 0, 1, 2, 3, 4. | `Tier 0` → T0 |
| **Changelog URL** | Link to the app's API changelog | `https://airtable.com/developers/web/api/changelog` |

### Example message

```text
The airtable app (Tier 0) wasn't checked for API changes or deprecations. This one needs a manual check.
Changelog
https://airtable.com/developers/web/api/changelog
```

**Extracted input for API Check Up:**

- **App** = `airtable` (component)
- **Tier** = T0
- **Changelog URL** = `https://airtable.com/developers/web/api/changelog`

### When to use Slack input

| When | Action |
|------|--------|
| **New message in #apps-api-checkups** | Treat as trigger: parse the message, extract app/tier/changelog URL, then run the API Check Up workflow. |
| **User pastes a message** | Same: parse and extract, then run the workflow for that app. |

---

## Airtable input

Simple overview of the Airtable base used by the API Check Up process. One base, two main spaces.

### Base

| Item | Value |
|------|--------|
| **Base ID** | `appuJmR3eUtJ8u4Yo` |
| **Use** | App ownership lookup (Resource Tracker) and Monitored apps (API check up data). |

### 1. App ownership (Resource Tracker)

**Purpose:** Resolve which **team** owns an app (component).

| Item | Value |
|------|--------|
| **Label** | Default space for app ownership |
| **URL** | [Resource Tracker view](https://airtable.com/appuJmR3eUtJ8u4Yo/tblFKFmjbDNFspUa1/viwCVNTyCISyyAzlL) |
| **Table ID** | `tblFKFmjbDNFspUa1` |

**Key field for ownership:**

| Field | Purpose |
|-------|---------|
| **team** | Value = **Athena** \| **Atlantis** \| **Plato** \| **Hermes**. Use this to assign correct team ownership for the app (component). |

**When to use:** Given an app/component name, look up the record and read **team** to detect correct team ownership (Athena, Atlantis, Plato, or Hermes).

### 2. Monitored apps (API check up overview)

**Purpose:** Store per-app API check up metadata and links.

| Item | Value |
|------|--------|
| **Label** | API check up space overview |
| **URL** | [Monitored apps (all records)](https://airtable.com/appuJmR3eUtJ8u4Yo/pagvvsUv8Cv3qCriU?0ORbv=allRecords) |

**Record fields (per app):**

| Field | Purpose |
|-------|---------|
| **API changelog / API Documentation** | URL(s) for changelog and API docs. |
| **app tier** | T0, T1, T2, T3, or T4 (check frequency). We recognize tiers 0, 1, 2, 3, 4. |
| **team** | Athena \| Atlantis \| Plato \| Hermes (ownership). |
| **isExternal** | `true` or `false`. |
| **App Name** | App/component identifier (for lookups and JIRA component). |
| **Last API Changelog Check Date** | When the app was last checked; update after each run. |

**When to use:** Look up app by name to get changelog URL, tier, team, isExternal; update Last API Changelog Check Date after processing.

---

## JIRA structure

Simple overview of how any new **[API Update]** ticket shall be structured, used when creating a ticket after detecting integration impact.

### Rule: Check for duplicates first

**Before creating any new [API Update] ticket**, always check for existing tickets with the same **[component]** (app label). Do not create a duplicate; instead update or link to the existing ticket as defined by the workflow.
Use reference of last 3 months in the past for the referenced tickets and duplicity checking.

### Title

| Format | Example |
|--------|---------|
| `[API Update] [component]: Summary of the changes` | `[API Update] airtable: Changelog updates require integration review` |

- **[component]** = app label (e.g. `airtable`, `openai-gpt-3`). Same identifier used in Slack message and Airtable.

### Description

Use these sections in the ticket description:

| Section | Content |
|---------|---------|
| **API Change Published Date** | Date when this API check up was done (analyzed and ticket created). Use only the date. |
| **API Due Date** | End of life / deprecation / shutdown date set by the 3rd party. **If empty:** see rule below. |
| **API Change Description** | Summary of the changes (from changelog/deprecation notice). |
| **Scope** | Format as a table with columns **Type** (Modules \| RPCs \| Functions) and **Affected** (names per type). One row per type. See example below. |
| **Code analysis summary** | Outcome of make-custom-app analysis: impact vs changelog/API docs (e.g. table: topic, impact, code change needed?), endpoints used, and conclusion or recommendations. Use heading **SDK API Check up — Code analysis**. |
| **What to do in the code** | **When code changes are required:** Concrete, actionable items (component/path, current behavior, required change). **When outcome = no change:** Skip the detailed table/list; state "No code change required" and optionally what was verified. |

**Scope table example:**

| Type     | Affected |
|----------|----------|
| Modules  | copyFileFolder, createFolder, getFile, ... |
| RPCs     | listFolders, listFiles, ... |
| Functions| pathRootJsonString, ... |

#### API Due Date when empty

| App tier | When API Due Date is empty |
|----------|----------------------------|
| **Tier 0 or 1** | Set to **current date + 30 days**. |
| **Tier 2, 3, or 4** | Leave **empty**. |

### JIRA fields

| Field | How to set |
|-------|------------|
| **Team** | From [Airtable input](#airtable-input): look up app by **app label / App Name**, read **team** value → **Athena** \| **Atlantis** \| **Plato** \| **Hermes**. |
| **Assignee** | Leave **empty**. |
| **Due date** | Set from **API Due Date** (after applying the tier rule above when API Due Date is empty). |
| **Component** | Set to the app (same as [component] in the title). |
