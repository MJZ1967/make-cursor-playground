---
name: sdk-api-checkup-jira
description: Defines how new [API Update] JIRA tickets shall look for the API Check Up process. Use when creating an API Update ticket, filling title/description/team/due date, or checking for duplicate tickets by component.
---

# JIRA structure for API Update tickets

Simple overview of how any new **[API Update]** ticket shall be structured. Used by the [API Check Up process](.cursor/skills/sdk-api-checkup/API%20Check%20Up%20task.md) when creating a ticket after detecting integration impact.

---

## Rule: Check for duplicates first

**Before creating any new [API Update] ticket**, always check for existing tickets with the same **[component]** (app label). Do not create a duplicate; instead update or link to the existing ticket as defined by the workflow.

---

## Title

| Format | Example |
|--------|---------|
| `[API Update] [component]: Summary of the changes` | `[API Update] airtable: Changelog updates require integration review` |

- **[component]** = app label (e.g. `airtable`, `openai-gpt-3`). Same identifier used in Slack message and Airtable.

---

## Description

Use these sections in the ticket description:

| Section | Content |
|---------|---------|
| **API Change Published Date** | Date when this API check up was done (analyzed and ticket created). Use only the date. |
| **API Due Date** | End of life / deprecation / shutdown date set by the 3rd party. **If empty:** see rule below. |
| **API Change Description** | Summary of the changes (from changelog/deprecation notice). |
| **Scope** | Format as a table with columns **Type** (Modules \| RPCs \| Functions) and **Affected** (names per type). One row per type. See example below. |
| **Code analysis summary** | Outcome of make-custom-app analysis: impact vs changelog/API docs (e.g. table: topic, impact, code change needed?), endpoints used, and conclusion or recommendations. Use heading **SDK API Check up — Code analysis**. |

**Scope table example:**

| Type     | Affected |
|----------|----------|
| Modules  | copyFileFolder, createFolder, getFile, ... |
| RPCs     | listFolders, listFiles, ... |
| Functions| pathRootJsonString, ... |

### API Due Date when empty

| App tier | When API Due Date is empty |
|----------|----------------------------|
| **Tier 0 or 1** | Set to **current date + 30 days**. |
| **Tier 2** | Leave **empty**. |

---

## JIRA fields

| Field | How to set |
|-------|------------|
| **Team** | From [Airtable](.cursor/skills/sdk-api-checkup/Airtable.md): look up app by **app label / App Name**, read **team** value → **Athena** \| **Atlantis** \| **Plato** \| **Hermes**. |
| **Assignee** | Leave **empty**. |
| **Due date** | Set from **API Due Date** (after applying the tier rule above when API Due Date is empty). |
| **Component** | Set to the app (same as [component] in the title). |

---

## Summary

- **Before create:** Always check for duplicates by **[component]**.
- **Title:** `[API Update] [component]: Summary of the changes`.
- **Description:** API Change Published Date (date of check up only), API Due Date (tier 0/1 → +30 days if empty; tier 2 → leave empty), API Change Description, Scope (table: Type, Affected), Code analysis summary (SDK API Check up — Code analysis).
- **Team** = from Airtable (app label → team). **Assignee** = empty. **Due date** = API Due Date.
- **Component** = app label.
