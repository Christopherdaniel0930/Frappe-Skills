---
name: frappe-project-planner
description: Analyze Frappe application requirements and create or maintain practical, dependency-aware implementation plans with tasks, priorities, ownership, milestones, and acceptance criteria. Use when planning a new Frappe app or substantial changes to an existing one; not for implementing a single isolated feature.
---

# Plan a Frappe Project

Turn the user's requirements into a clear, actionable Frappe development plan. Inspect the project before planning work, distinguish confirmed requirements from assumptions, and keep the plan proportional to the requested outcome. A plan is a coordination aid, not a reason to delay a small implementation the user has already asked you to perform.

## 1. Inspect the project

Before planning changes to an existing project:

- Identify whether the workspace contains a Frappe app, a bench, or only a project brief. Locate app metadata, modules, DocTypes, controllers, hooks, scripts, workflows, reports, fixtures, patches, and existing tests as relevant.
- Determine the Frappe version and installed app dependencies from the available source/configuration. Call out version uncertainty rather than assuming behavior.
- Review existing functionality and project conventions. Separate completed, partial, missing, and potentially duplicate work.
- Note environment constraints that affect the plan, such as no bench/site access, unavailable dependencies, or site-level actions requiring separate authorization.
- For a new app with no repository yet, plan only the setup and architecture detail supported by the brief; do not pretend to have inspected a project.

## 2. Analyze requirements

Summarize the goal and identify the requirements that affect implementation:

- Business objectives and user workflows.
- User roles, access expectations, and security needs.
- Modules, DocTypes, relationships, fields, lifecycle behavior, and business rules.
- Approval workflows, integrations, APIs, scheduled work, reports, dashboards, and UI needs.
- Data migration, deployment, operational, performance, and compatibility constraints when relevant.
- Mandatory scope versus optional ideas.

Do not invent business behavior. Record assumptions explicitly and flag only ambiguities that block a sound plan or materially change access, data model, lifecycle, or scope. Continue planning independent work while noting such blockers.

## 3. Break scope into modules and deliverables

Group the requested capabilities into logical modules or vertical slices. For each, state its responsibility and dependencies. Avoid creating separate modules or phases just to match a template. Do not add optional features, integrations, or infrastructure tasks without a requirement or clear necessity.

Prefer tasks that deliver usable increments. For example, a booking feature may require a DocType, validation, permissions, and a form workflow; keep these distinct when ownership/dependencies differ, but preserve their relationship in the plan.

## 4. Create actionable tasks

Give each task:

- A stable ID (for example, `TASK-001`); retain existing IDs when updating a plan.
- A concise name and concrete outcome.
- Module or workstream.
- Priority and complexity, with brief meaning if the audience needs it.
- Prerequisites and explicit task dependencies.
- Owner/role when useful and supported by the context.
- Expected deliverable and measurable acceptance criteria.
- Status, blocker, and effort estimate when requested or useful.

Task quality rules:

- Phrase tasks as outcomes that can be reviewed, not vague activities such as “work on backend.”
- Keep tasks independently understandable and small enough to estimate or assign.
- Split work when distinct expertise, sequencing, or deliverables warrant it; avoid excessive fragmentation.
- Make acceptance criteria observable. Do not use “works correctly” without specifying what should be verified.
- Estimate effort as a range or relative size when uncertainty is high. Label estimates as assumptions, not commitments.

Suggested status vocabulary: `Not Started`, `Ready`, `In Progress`, `Blocked`, `Completed`, `Failed`. Use only when maintaining progress; a proposed plan can mark tasks as `Ready` or `Not Started` consistently. Never mark work completed without evidence.

Suggested priority meanings:

| Priority | Meaning |
|---|---|
| Critical | Blocks core operation, correctness, or required delivery |
| High | Needed for the primary user workflow |
| Medium | Supporting scope or important operational improvement |
| Low | Optional enhancement that can be deferred |

Suggested complexity meanings:

| Complexity | Meaning |
|---|---|
| Low | Bounded change with few dependencies |
| Medium | Several coordinated steps or nontrivial validation |
| High | Significant uncertainty, cross-module work, or complex behavior |

## 5. Model dependencies and execution order

- Add an edge from a prerequisite task to every task that depends on it.
- Ensure DocType and data model decisions precede dependent controller, workflow, report, or integration work.
- Ensure business rules and access decisions are clear before implementation that relies on them.
- Put integration work after its required interfaces exist, unless parallel interface work is explicitly coordinated.
- Identify blockers caused by missing decisions, access, or external systems.
- Detect cycles and resolve them by clarifying interfaces or splitting tasks.
- Identify tasks that can proceed in parallel only when they do not compete for the same files, metadata, or decisions.

Show the ordering in a dependency column, numbered phases, or a compact graph. Use the simplest representation that makes sequencing clear.

## 6. Assign ownership and delegation carefully

Suggest a role or specialist for each task only when it improves coordination. Example responsibilities include DocType/data modeling, business logic, workflows, Server Scripts, client behavior, reports, integration, and validation.

- Treat labels such as `DocType Architect` or `Workflow Specialist` as suggested responsibilities, not as real agents or guaranteed capabilities.
- Delegate work only when the user requests delegation or the environment explicitly makes relevant agents available and delegation is appropriate. Do not imply an assignment has happened when it is only a plan.
- Keep a single owner for each deliverable and identify handoffs through dependencies.
- For work that spans specialties, split deliverables at clear interfaces and sequence them explicitly.
- Avoid parallel assignments that would cause conflicting edits to shared metadata or files.

## 7. Organize phases and milestones

Use phases that reflect this project's needs. A common sequence is:

1. Requirements and constraints.
2. Data model and architecture.
3. Core user workflow and business logic.
4. Permissions, approvals, integrations, and automation.
5. User interface and reporting.
6. Verification, deployment preparation, and documentation.

Combine, reorder, or omit phases as appropriate. Define milestones as reviewable outcomes, not dates unless dates or schedule constraints were provided. Identify milestone prerequisites and what evidence indicates completion.

## 8. Plan validation and quality

Before presenting the plan, check that:

- Every mandatory requirement maps to one or more tasks.
- Tasks do not duplicate existing or completed functionality.
- Each task has a concrete deliverable and acceptance criteria.
- Dependencies are valid, acyclic, and ordered correctly.
- Ownership suggestions match the responsibility and do not imply unavailable agents.
- Priorities reflect user value, blocking relationships, and risk.
- Optional scope is labeled and can be deferred.
- Verification tasks cover the requested behavior, permissions, lifecycle, integrations, and regressions as applicable.
- Environment/site actions such as install, migrate, activate, deploy, or modify production data are listed separately and are not treated as automatically authorized.

Do not add or run tests solely because a plan mentions testing; testing actions follow the user's request and project instructions.

## 9. Maintain an existing plan

When requirements or progress change:

- Preserve stable IDs and completed work that remains valid.
- Identify affected tasks, dependencies, criteria, estimates, and milestones.
- Add, revise, defer, or remove tasks narrowly; explain significant changes.
- Recompute readiness and execution order after dependency changes.
- Record blockers and the specific evidence needed to clear them.
- Mark a task completed only when its criteria have verification evidence. Record the evidence briefly.
- Distinguish planned, in-progress, implemented, and verified work.

## 10. Present the plan

Choose the format that best fits project size. For a substantial plan, include:

1. Goal and scope summary.
2. Project context and assumptions.
3. Modules or workstreams.
4. Phases or milestones.
5. Task table with ID, deliverable, owner/role, priority, complexity, dependencies, status, and acceptance criteria.
6. Execution order and safe parallel work.
7. Estimates, if useful, with uncertainty stated.
8. Risks, blockers, and decisions needed.

For a small change, provide a short ordered checklist instead of manufacturing a project-management document. Make the next actionable task clear.

## Critical rules

- Never invent requirements or present assumptions as user decisions.
- Never plan duplicate work without explaining why existing functionality must change.
- Never ignore or misorder dependencies.
- Never assign work to an agent as if delegation occurred when it did not.
- Never mark unverified work complete.
- Never turn optional enhancements into required scope.
- Never treat implementation, testing, migration, activation, and deployment as the same action.
- Update the plan when requirements or verified progress change.
