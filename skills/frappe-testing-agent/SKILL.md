---
name: frappe-testing-agent
description: Plan and, when explicitly requested, execute tests for Frappe application behavior, business rules, workflows, permissions, APIs, integrations, and UI. Use to verify implemented functionality and report reproducible findings; use frappe-post-creation-validation for static metadata and artifact checks.
---

# Test Frappe Applications

Verify Frappe behavior against user requirements and measurable acceptance criteria. Inspect the installed version, app, site, existing test conventions, and environment before choosing test methods. Report only evidence actually obtained. This skill covers behavioral and runtime testing; static schema/configuration review belongs to the post-creation validation skill and can complement tests.

## 1. Inspect the target and environment

- Identify the Frappe version, app, target site/environment, installed apps, and implementation under test.
- Review relevant DocTypes, controllers, APIs, Client Scripts, Server Scripts, workflows, hooks, jobs, and existing tests.
- Inspect existing test fixtures and conventions before creating or changing tests.
- Confirm the target is a safe test environment. Frappe tests should run on an appropriately configured test site (commonly named with a `test_` prefix); verify the installed version's safeguards and `allow_tests` configuration.
- Check required bench, site, dependencies, services, credentials, and external test doubles. Do not infer that workers, scheduler, browser automation, or external services are available.
- Do not run tests unless the user explicitly asks to test or verify. A request to create a skill, review code, or implement a feature alone is not authorization to run tests.

## 2. Derive testable expectations

Use user requirements, acceptance criteria, and implementation specifications as the source of expected behavior. For each feature, record:

- Behavior and expected result.
- Inputs, preconditions, and relevant document state.
- Validation and boundary conditions.
- Expected failure mode and error behavior.
- Required user role and permission outcome.
- Dependencies, side effects, and related behavior to regress.

Do not invent expected business behavior. Ask about consequential ambiguity when it changes correctness, permissions, lifecycle, or expected output; otherwise state the assumption and limit conclusions accordingly.

## 3. Select an appropriate test level

| Test level | Use for |
|---|---|
| Unit | A bounded function or method, with dependencies controlled |
| Frappe document/integration | ORM behavior, controller hooks, validation, permissions, lifecycle |
| API | Authentication, input validation, permissions, response and errors |
| Workflow | State/action availability, roles, transitions, DocStatus behavior |
| UI | Browser form interactions and user journeys |
| Job/scheduler | Enqueueing, task behavior, retries, idempotency and recovery |
| Integration | Contract with an external provider, preferably using mocks or a sandbox |
| Regression | Previously working or adjacent behavior affected by the change |
| Performance/security | Specific risks requiring suitable controlled methods and scope |

Choose the narrowest useful tests and add broader coverage for high-impact paths. Do not claim a UI, security, performance, or end-to-end property based solely on unit tests or static inspection.

## 4. Build a practical test plan

For each important behavior, include valid cases and relevant invalid, boundary, missing-value, duplicate, and permission cases. Consider concurrency or retries when they can affect correctness. Avoid redundant test cases that do not provide distinct evidence.

Examples of areas to cover when in scope:

- DocType required fields, relationships, defaults, naming, uniqueness, and lifecycle.
- Business rule calculations, date/number boundaries, and cross-document constraints.
- Workflow initial state, permitted and forbidden transitions, roles, rejection/reopen behavior, and submission/cancellation.
- API inputs, authentication, permission checks, response shape, and error handling.
- Client behavior in a real browser: events, fields, buttons, filters, dialogs, and child rows.
- Background job status from enqueue through completion/failure; repeat execution and recovery.
- Webhook signatures, malformed input, replay, and duplicate delivery.
- Reports' filters, aggregates, and permission-scoped records.

Define a clear expected result for every test. The UI hiding an action is not evidence of server-side authorization; test authorization through Frappe's actual permission mechanisms.

## 5. Use Frappe's test infrastructure appropriately

- Follow the installed version's testing docs and repository patterns. Frappe commonly uses `FrappeTestCase` for framework-aware Python tests and Bench `run-tests` commands, but import paths, flags, and CLI options can vary.
- Confirm command syntax from the installed Bench/Frappe version before running it. Do not copy a sample site name or assume the full app suite is the right scope.
- Use isolated, deterministic test records and clean them up through supported test patterns. Prefer generated unique values and fixtures that are safe to reuse.
- Mock outbound HTTP, email, payment, and other external side effects for ordinary automated tests. Use provider sandboxes or real requests only when explicitly authorized and configured.
- Do not use production data or run destructive cleanup. Never reset or recreate a database to make tests pass.
- Keep tests repeatable and independent of record ordering, current time, locale, and external service availability where practical.

When test implementation is requested, add focused tests that match the existing project conventions. When execution is requested, run only the relevant tests first and expand scope based on results and risk.

## 6. Test Frappe behavior accurately

### Documents and business rules

- Test through the public document lifecycle (insert/save/submit/cancel) appropriate to the behavior, rather than directly changing database rows and bypassing validations.
- Include permission and document status conditions. Verify server-side enforcement, not just client feedback.
- Assert meaningful values and exceptions, not merely that a function returned something non-empty.

### Workflows and permissions

- Verify valid actions succeed and unauthorized or invalid actions fail.
- Check workflow state and `docstatus` separately: draft `0`, submitted `1`, cancelled `2`.
- Test submission/cancellation/amendment constraints for submittable DocTypes and role-based access.

### Jobs, hooks, and integrations

- Distinguish queued, started, completed, and failed jobs; enqueue success does not prove job completion.
- Exercise duplicate/retry paths and ensure idempotency where side effects could repeat.
- Test transaction timing and after-commit assumptions where applicable.
- Keep external calls mocked unless sandbox or production calls are explicitly approved.

### UI

- Use the project's supported browser/UI test framework when available.
- Verify behavior through the rendered Desk or Web UI for UI claims. JavaScript syntax checks do not verify form behavior, responsiveness, or accessibility.
- Cover representative roles, document states, empty/error states, and relevant viewport sizes when the requirement calls for them.

## 7. Run tests safely when authorized

Before execution:

- Confirm the exact command and site target; verify that the site is a test site and that test execution is enabled.
- Review whether tests may send emails, enqueue jobs, call external services, modify shared fixtures, or make destructive changes.
- Confirm required dependencies and service processes are available.
- Ask for authorization if execution would touch production, cause external side effects, or require a risky environment change.

Run the narrowest relevant command first. Capture the exact command, exit status, and concise output. If a test is blocked by missing dependencies or environment, report that rather than disguising it as a code failure. Do not change production configuration, migrate, restart workers, or send real third-party requests as an incidental part of testing.

## 8. Classify and report results

Use consistent statuses:

| Status | Meaning |
|---|---|
| Passed | Test was executed and matched its expected result |
| Failed | Test was executed and behavior differed from expectation |
| Blocked | Could not execute because an environment/dependency prerequisite was unavailable |
| Skipped | Intentionally not run, with reason |
| Not Run | No execution attempted |
| Needs Review | Evidence is ambiguous or requires investigation |

For failures, report:

- Test name or scenario and affected feature.
- Preconditions and concise reproduction steps.
- Expected and actual results.
- Relevant error/traceback, sanitized for secrets and personal data.
- Severity based on impact, and likely cause only when evidence supports it.
- Whether implementation was changed and what remains to verify.

Do not silently change implementation while acting only as tester. If a fix is requested, separate the fix from the test finding and rerun relevant tests only when authorized.

Summarize scope, environment, tests actually executed, results, failures, blocked/skipped areas, and limitations. Keep static review findings separate from test execution results.

## Critical rules

- Never claim an unexecuted test passed.
- Never invent expected results or mark blocked/skipped tests as passed.
- Never run tests without explicit user request.
- Never run destructive tests against production or shared data.
- Never send real external requests without authorization.
- Never expose secrets or sensitive records in test output.
- Always test critical business rules and permissions on the server.
- Always distinguish static inspection from runtime verification.
- Always report the actual scope and limits of testing.