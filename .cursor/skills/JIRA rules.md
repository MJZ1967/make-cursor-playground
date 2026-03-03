---
name: jira-rules
description: Defines and applies JIRA rules, workflows, best practices, and ticket handling. Use when defining project JIRA conventions, creating workflows, writing ticket templates, establishing best practices, or when the user asks about JIRA setup, ticket lifecycle, or issue tracking standards.
---

# JIRA Rules, Workflows & Best Practices

Skill for defining and applying JIRA rules, workflows, best practices, and ticket handling in projects.

---

## 1. JIRA Rules Definition

### 1.1 When to Define Rules

- New project or board setup
- Team or org standardization
- Compliance or audit requirements
- Reducing noise and inconsistency in tickets

### 1.2 Rule Categories

| Category | Purpose | Examples |
|----------|---------|----------|
| **Naming** | Consistent issue keys and summaries | `PROJ-123`, `PROJ-123-short-description` |
| **Required fields** | What must be set before transition | Assignee, Sprint, Story Points, Labels |
| **Validation** | Block invalid transitions or empty fields | "Cannot move to Done without Resolution" |
| **Automation** | Auto-assign, auto-label, notifications | Assign to component lead when created |
| **Permissions** | Who can do what per project/role | Only QA can close bugs |

### 1.3 Rule Syntax Conventions

- Prefer **explicit over implicit**: state the condition and the action.
- Use **positive rules** where possible (e.g. "Require X" instead of "Reject when not X").
- Document **exceptions** (e.g. "Except for Epic type").

---

## 2. Workflows

### 2.1 Workflow Structure

- **States**: e.g. To Do → In Progress → In Review → Done (adapt to team).
- **Transitions**: name clearly (e.g. "Start progress", "Submit for review", "Reopen").
- **Conditions**: who/what can trigger each transition.
- **Validations**: what must be true before transition (fields, links, subtasks).
- **Post-functions**: what runs after transition (notifications, field updates, automation).

### 2.2 Common Workflow Patterns

- **Linear**: To Do → In Progress → Done.
- **With review**: To Do → In Progress → In Review → Done.
- **With QA**: To Do → In Progress → Ready for QA → In Review → Done.
- **Reopen path**: explicit "Reopen" from Done/Closed back to To Do or In Progress.

### 2.3 Workflow Best Practices

- Keep number of statuses manageable (typically 4–7).
- Avoid duplicate or overlapping statuses.
- Document expected use of each status and transition.
- Use screen schemes so only relevant fields show per status where possible.

---

## 3. Best Practices

### 3.1 Ticket Quality

- **Summary**: One clear action/outcome; include ticket key when useful.
- **Description**: Context, acceptance criteria, and links (design, docs, related tickets).
- **Labels**: Consistent set (e.g. `team-x`, `tech-debt`, `priority-high`).
- **Links**: Use "blocks", "is blocked by", "relates to" to keep dependencies visible.

### 3.2 Sizing & Estimation

- Use **Story Points** or **T-shirt sizes** consistently per project.
- Definition of "Done" aligned across team (code, tests, docs, review).
- Avoid oversized tickets; split when exceeding agreed max size.

### 3.3 Board Hygiene

- Limit WIP per status or per person where it helps.
- Regular backlog refinement and prioritization.
- Archive or close obsolete tickets; keep active sprints focused.

### 3.4 Naming & Conventions

- **Commit messages**: reference ticket key (e.g. `PROJ-123 Add login validation`).
- **Change references**: include ticket key and short summary in deploys or change records.

---

## 4. Ticket Handling

### 4.1 Lifecycle

1. **Create**: Correct type (Story, Bug, Task, Epic), project, component.
2. **Triage**: Priority, assignee (or leave unassigned per process), sprint/backlog.
3. **Refine**: Acceptance criteria, estimate, dependencies.
4. **Progress**: Move through workflow; keep status and assignee up to date.
5. **Complete**: Resolution, time tracking if used, link to deploy.
6. **Close/Archive**: When done and no longer needed on the board.

### 4.2 Ticket Types

| Type | Use when |
|------|----------|
| **Epic** | Large initiative or theme; contains Stories/Tasks |
| **Story** | User-facing value; has acceptance criteria |
| **Task** | Work without direct user story (e.g. tech task, spike) |
| **Bug** | Something is broken or wrong behavior |
| **Subtask** | Part of a parent Story/Task |

### 4.3 Handling Blocked Items

- Use "blocked by" link or label (e.g. `blocked`).
- Optional status "Blocked" or clear convention (e.g. comment + label).
- Unblock when dependency is resolved; update status and remove block.

### 4.4 Templates

- **Story**: Goal, context, acceptance criteria, out of scope, links.
- **Bug**: Steps to reproduce, expected vs actual, environment, severity.
- **Task**: Objective, steps or checklist, definition of done.

---

## 5. DONE Tickets & Continuous Release

Best practices and release guidelines for DONE tickets in a continuous release process.

### 5.1 Definition of Done for Release

Before a ticket can be considered releasable from DONE:

- **Code**: Integrated and included in the release stream.
- **Review**: All CI checks passing.
- **Tests**: Unit, integration, and E2E where applicable; no known regressions.
- **Documentation**: Changelog, runbooks, or API docs updated if required.
- **No blockers**: No linked "blocked by" or open critical bugs.
- **Resolution**: Properly resolved (e.g. Fixed, Done, Deployed) per project convention.

### 5.2 Release Conditions

| Condition | Requirement |
|-----------|-------------|
| **Deployability** | No broken builds; deployment pipeline green |
| **Feature flags** | Optional: work behind flag until ready; document flag name |
| **Dependencies** | Linked tickets or services deployed in correct order |
| **Rollback readiness** | Rollback plan or feature flag off-switch documented where relevant |

### 5.3 Best Practices for Continuous Release

- **Small, frequent releases**: Prefer smaller batches of DONE tickets; avoid large release trains.
- **Traceability**: Link commits and deploys to ticket keys (e.g. `PROJ-123`).
- **Release notes**: Use labels like `release-notes` or `changelog` to flag tickets for release notes.
- **Environment tracking**: Optional custom field for "Deployed to" (e.g. Staging, Production) if not automated.
- **Hotfix handling**: Use separate workflow path for hotfixes (e.g. direct to production); document in ticket.

### 5.4 Post-DONE Workflow

- **Deployed**: Optional status or label when work is live in production.
- **Monitoring**: Link to dashboards, alerts, or runbooks for monitoring new changes.
- **Feedback loop**: Reopen if production issues arise; create linked Bug ticket for tracking.
- **Archiving**: Close only after stable in production for agreed period (e.g. 7 days).

### 5.5 Release Validation Checklist

Before including a DONE ticket in a release:

- [ ] Code integrated in the release stream.
- [ ] CI/CD pipeline green.
- [ ] Tests passing; no known critical issues.
- [ ] Documentation and changelog updated.
- [ ] Feature flags configured (if used).
- [ ] Dependencies and order of deployment verified.
- [ ] Rollback or feature-disable option documented where needed.

---

## 6. Quick Reference Checklist

When defining or reviewing JIRA setup:

- [ ] Workflow states and transitions are documented and understood.
- [ ] Required fields and validations are set for key transitions (e.g. to Done).
- [ ] Naming and commit conventions are written down.
- [ ] Ticket types and when to use each are clear.
- [ ] Templates exist for Story, Bug, and Task (or equivalent).
- [ ] Board filters and columns match the workflow and team usage.
- [ ] Automation rules are documented (what runs when).
- [ ] Definition of Done and release conditions for continuous release are defined.

---

## 7. Additional Resources

- Link to internal JIRA space or Confluence page for project-specific rules.
- Link to workflow diagrams or screenshots if available.
- Reference to team SLA or response expectations (e.g. triage within 24h).
