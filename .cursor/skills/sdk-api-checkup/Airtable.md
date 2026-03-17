---
name: sdk-api-checkup-airtable
description: Defines Airtable spaces and fields for the API Check Up process. Use when resolving app team ownership (Resource Tracker), when reading or updating Monitored apps (changelog, tier, team, isExternal), or when explaining Airtable input to the API Check Up workflow.
---

# Airtable input for API Check Up

Simple overview of the Airtable base used by the [API Check Up process](.cursor/skills/sdk-api-checkup/API%20Check%20Up%20task.md). One base, two main spaces.

---

## Base

| Item | Value |
|------|--------|
| **Base ID** | `appuJmR3eUtJ8u4Yo` |
| **Use** | App ownership lookup (Resource Tracker) and Monitored apps (API check up data). |

---

## 1. App ownership (Resource Tracker)

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

---

## 2. Monitored apps (API check up overview)

**Purpose:** Store per-app API check up metadata and links.

| Item | Value |
|------|--------|
| **Label** | API check up space overview |
| **URL** | [Monitored apps (all records)](https://airtable.com/appuJmR3eUtJ8u4Yo/pagvvsUv8Cv3qCriU?0ORbv=allRecords) |

**Record fields (per app):**

| Field | Purpose |
|-------|---------|
| **API changelog / API Documentation** | URL(s) for changelog and API docs. |
| **app tier** | T0, T1, or T2 (check frequency). |
| **team** | Athena \| Atlantis \| Plato \| Hermes (ownership). |
| **isExternal** | `true` or `false`. |
| **App Name** | App/component identifier (for lookups and JIRA component). |
| **Last API Changelog Check Date** | When the app was last checked; update after each run. |

**When to use:** Look up app by name to get changelog URL, tier, team, isExternal; update Last API Changelog Check Date after processing.

---

## Summary

- **Base ID:** `appuJmR3eUtJ8u4Yo`
- **Resource Tracker** (app ownership): use **team** → Athena / Atlantis / Plato / Hermes for component ownership.
- **Monitored apps** (API check up): store and read **API changelog/docs**, **app tier**, **team**, **isExternal**, and **Last API Changelog Check Date**.
- **Next step:** Use with [Slack](.cursor/skills/sdk-api-checkup/Slack.md) input and the [API Check Up task](.cursor/skills/sdk-api-checkup/API%20Check%20Up%20task.md) (changelog check, impact analysis, JIRA).
