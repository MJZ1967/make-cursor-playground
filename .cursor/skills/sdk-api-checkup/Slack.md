---
name: sdk-api-checkup-slack
description: Defines where and what to look for in Slack when triggering the API Check Up process. Use when watching the channel for new messages, parsing a message to get app/changelog/tier, or explaining the Slack input to the API Check Up workflow.
---

# Slack input for API Check Up

Simple overview of the Slack source used by the [API Check Up process](.cursor/skills/sdk-api-checkup/API%20Check%20Up%20task.md).

---

## Channel

| Item | Value |
|------|--------|
| **Channel** | `#apps-api-checkups` |
| **Purpose** | One message per app = one component that needs an API/changelog check. |

---

## Message = one app (component)

- **One message** in the channel = **one app** = **one component**.
- The app is represented as a **block of code** in the message: `[app]` (e.g. code block or similar delimiter in the actual message).
- Use the content of that block as the **app identifier** (e.g. `airtable`, `openai-gpt-3`).

---

## What to look for in each message

| Field | What to extract | Example |
|-------|------------------|--------|
| **App name** | From the message text or the `[app]` block | `airtable` |
| **Tier** | T0 / T1 / T2 (in parentheses in the text) | `Tier 0` → T0 |
| **Changelog URL** | Link to the app’s API changelog | `https://airtable.com/developers/web/api/changelog` |

---

## Example message

```text
The airtable app (Tier 0) wasn’t checked for API changes or deprecations. This one needs a manual check.
Changelog
https://airtable.com/developers/web/api/changelog
```

**Extracted input for API Check Up:**

- **App** = `airtable` (component)
- **Tier** = T0
- **Changelog URL** = `https://airtable.com/developers/web/api/changelog`

---

## When to use this

| When | Action |
|------|--------|
| **New message in #apps-api-checkups** | Treat as trigger: parse the message, extract app/tier/changelog URL, then run the [API Check Up workflow](.cursor/skills/sdk-api-checkup/API%20Check%20Up%20task.md). |
| **User pastes a message** | Same: parse and extract, then run the workflow for that app. |
| **User asks “what does the Slack message contain?”** | Use this doc as the overview of channel, message format, and fields to extract. |

---

## Summary

- **Channel:** `#apps-api-checkups`
- **One message** = one app (component); app id from message / `[app]` block.
- **Extract:** App name, Tier (T0/T1/T2), Changelog URL.
- **Next step:** Pass extracted data into the [API Check Up task](.cursor/skills/sdk-api-checkup/API%20Check%20Up%20task.md) (Airtable lookup, changelog check, impact analysis, JIRA).
