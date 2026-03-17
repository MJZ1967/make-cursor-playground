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
| **API Change Published Date** | Date the 3rd party published the API change. |
| **API Due Date** | End of life / deprecation / shutdown date set by the 3rd party. **If empty:** see rule below. |
| **API Change Description** | Summary of the changes (from changelog/deprecation notice). |
| **Scope** | List of modules, RPCs, or functions affected by the changes. |

### API Due Date when empty

| App tier | When API Due Date is empty |
|----------|----------------------------|
| **Tier 0 or 1** | Set to **current date + 30 days**. |
| **Tier 2 or 3** | Leave **empty**. |

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
- **Description:** API Change Published Date, API Due Date (tier 0/1 → +30 days if empty; tier 2/3 → leave empty), API Change Description, Scope.
- **Team** = from Airtable (app label → team). **Assignee** = empty. **Due date** = API Due Date.
- **Component** = app label.
