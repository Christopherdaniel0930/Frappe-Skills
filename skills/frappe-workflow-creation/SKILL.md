---
name: frappe-workflow-creation
description: Design, validate, and implement native Frappe Workflows from business requirements, including states, actions, role restrictions, conditions, DocStatus behavior, and app metadata. Use when building or changing approval and document lifecycle workflows in a Frappe codebase; not for general process design or workflow questions unrelated to Frappe.
---

# Create a Frappe Workflow

Design and implement the smallest complete native Frappe Workflow that matches the user's business process. Treat workflow state and document lifecycle status as related but distinct concepts. Follow the installed Frappe version and the target app's existing conventions.

## Establish project context

- Inspect the repository, app, module, target DocType, existing workflows, fixtures, patches, and nearby metadata before editing.
- Identify the installed Frappe version from the bench, dependency declarations, lock files, or framework source. Use version-matched documentation and source when behavior or metadata fields are uncertain.
- Confirm the target DocType exists, whether it is submittable, how it is named, and whether it already has a workflow state field or an active workflow.
- Check how this app stores Workflow records: standard fixtures, exported DocType JSON, patches, or another established convention. Keep changes consistent with that approach.
- Do not assume ERPNext is installed. A Frappe app can define workflows for its own DocTypes.

## Turn requirements into a workflow design

Extract and record:

- Target DocType and the business purpose of the workflow.
- Initial state, intermediate review states, and intended terminal states.
- Actions users can take from each state.
- Roles allowed to perform each action.
- Conditions for transitions, including any required data or business rules.
- Whether approval submits the document, and whether rejection, cancellation, amendment, or reopening is allowed.
- Who may edit a document in each state and what happens after the terminal state.

Resolve only consequential ambiguities with the user. Do not silently invent approval authority, business conditions, or lifecycle behavior. Present a compact state and transition mapping before implementation when the requirements are substantial.

## Model states and DocStatus correctly

Frappe's document statuses are `0` (Draft), `1` (Submitted), and `2` (Cancelled). A workflow state is a business label stored in the configured workflow state field; it is not itself a DocStatus.

- Use `doc_status: 0` for states in which the document remains a draft, such as Draft, Pending Review, or Rejected when it has not been submitted.
- Use `doc_status: 1` for states that represent a submitted document. Make sure the target DocType supports submission and that the transition follows Frappe's submit lifecycle.
- Use `doc_status: 2` only for genuine cancellation through Frappe's cancellation lifecycle. Rejection is ordinarily a business state, not cancellation.
- Do not assume that an "Approved" state must be submitted. Map it to `1` only when the requirement and DocType lifecycle call for submission.
- Do not model a submitted document as returning to draft through an ordinary workflow transition. Follow Frappe's amendment, cancellation, and resubmission rules for the installed version.
- Validate every state's status mapping and every transition that crosses draft, submitted, or cancelled status.

Example only; adapt to requirements:

| State | Purpose | DocStatus |
|---|---|---:|
| Draft | Author is preparing the document | 0 |
| Pending Approval | Waiting for a decision | 0 |
| Approved | Approved and submitted, if required | 1 |
| Rejected | Decision declined; may be revised if allowed | 0 |
| Cancelled | Frappe cancellation completed | 2 |

## Design the transition graph

For every transition define source state, action label, destination state, allowed role, and any condition. Example:

| From | Action | To | Allowed role | Condition |
|---|---|---|---|---|
| Draft | Submit for Approval | Pending Approval | Employee | Required fields are complete |
| Pending Approval | Approve | Approved | Manager | None |
| Pending Approval | Reject | Rejected | Manager | Rejection reason is present |
| Rejected | Revise | Draft | Employee | Document is still a draft |

- Ensure source and destination states exist and every action is clear to its intended user.
- Ensure only the appropriate role can perform each transition. Grant only the roles required by the business process.
- Identify initial and terminal states explicitly. Check that every required state is reachable and every intended approval path has a valid outcome.
- Flag dead ends, unreachable states, duplicate or contradictory transitions, and cycles that could enable unintended repeated approvals.
- A terminal business state need not have an outgoing transition. If reopening is required, define who can do it and how it fits the DocStatus lifecycle.
- Use transition conditions for server-evaluable business rules supported by the installed version. Put complex or authoritative validation in server-side controller logic; do not rely on client-side checks for security.

## Roles, editing, and permissions

- Configure transition roles according to responsibility. Workflow roles and DocType permissions work together; verify that users can access the document and that the workflow limits the available actions as intended.
- Configure each state's edit allowance according to the installed version's Workflow metadata conventions. In versions where `allow_edit` is role-based, set the intended role explicitly and verify the role can edit only in the intended state.
- Do not assume workflow transitions replace DocType permissions, document sharing, controller validation, or other access controls.
- Protect the workflow state field from ordinary direct editing where the framework and project conventions allow it. State changes should occur through valid workflow actions.
- Avoid custom scripts that duplicate native workflow behavior. Add server-side validation only for rules that native metadata cannot safely enforce.

## Configure workflow metadata

Use only properties supported by the installed Frappe version. A typical Workflow record includes:

- `workflow_name`, `document_type`, `is_active`, and `workflow_state_field`.
- Workflow state rows with state name, `doc_status`, and applicable edit role or state properties.
- Workflow transition rows with source `state`, `action`, `next_state`, `allowed` role, and optional condition.
- Optional email alerts or other supported properties only when requested or needed.

The configured `workflow_state_field` must exist on the target DocType and be appropriate for storing the workflow state. If missing, add it using the app's normal approach (app-owned DocType field, fixture, or Custom Field configuration). Do not modify unrelated DocType metadata.

Persist the Workflow using the project's established convention. For a standard app, this may be a Workflow JSON file and fixture registration; for other projects, follow their existing export or patch pattern. A file in the app does not activate or install the workflow on a site by itself.

## Validate before implementation

Check the full design and resulting metadata:

- The Workflow targets the correct DocType and uses an existing state field.
- State names are unique; each has a valid DocStatus and appropriate edit behavior.
- Each transition references existing states, defines an action and allowed role, and uses a valid condition format for this Frappe version.
- The initial state, required paths, terminal states, rejection behavior, and reopening behavior are coherent.
- No required state is unreachable; unintended dead ends, cycles, or bypasses are identified and corrected.
- DocStatus transitions respect whether the DocType is submittable and follow Frappe's submit/cancel/amend lifecycle.
- Roles, DocType permissions, field permissions, and server validations jointly enforce the intended access.
- JSON is valid and metadata follows neighboring project conventions.

## Implement and report

- Create or update only the required workflow metadata and state field configuration.
- Avoid activating a workflow, running migrations, installing apps, or changing site data unless the user explicitly requested that operation. Keep file implementation distinct from site activation and migration steps.
- Do not add or run tests unless the user asks to test or verify implementation. If asked, use the project's existing validation or test workflow and report the result.
- Summarize the target DocType, states and DocStatus mapping, transitions and roles, conditions, files changed, assumptions, and any commands needed to export, migrate, or activate the workflow on a site.
- Clearly state which validation was performed and which site-level steps remain.

## Expected design output

For a workflow design or implementation, provide:

1. Target DocType and workflow purpose.
2. State definitions, including initial/terminal designation and DocStatus mapping.
3. Transition graph with actions, roles, and conditions.
4. Editing, rejection, cancellation, and reopening rules.
5. Permission and lifecycle validation findings.
6. Workflow metadata and any required state field configuration.
7. Implementation files and site-level follow-up steps.

## Critical rules

- Never define a transition without a valid source state, destination state, action, and authorized role.
- Never use DocStatus `2` for an ordinary rejection.
- Never treat business state labels as substitutes for Frappe DocStatus.
- Never assume approval implies submission; establish that from the requirement and DocType lifecycle.
- Never bypass Frappe's submission, cancellation, or amendment lifecycle.
- Always inspect version and project conventions, then validate the complete transition graph before implementation.
