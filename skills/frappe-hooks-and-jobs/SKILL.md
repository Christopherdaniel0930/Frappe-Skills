---
name: frappe-hooks-and-jobs
description: Design and implement Frappe application hooks, document events, scheduled tasks, background jobs, webhooks, and external integrations. Use when adding app-level automation or integration behavior to a Frappe codebase; not for general Python job queues unrelated to Frappe.
---

# Frappe Hooks, Jobs, and Integrations

Implement reliable Frappe automation using the installed framework version and the app's existing architecture. Separate hook registration from business logic, distinguish synchronous document lifecycle work from asynchronous jobs, and design external operations for retries, duplicate delivery, and failure recovery.

## 1. Inspect the environment

- Identify the Frappe version, app and module structure, installed dependencies, and target site/bench context if available.
- Inspect existing `hooks.py`, document controllers, event handlers, integrations, scheduled jobs, and tests. Reuse existing services and avoid duplicate hook registrations.
- Confirm APIs and hook names against version-matched docs or installed framework source. Do not assume a hook, event signature, scheduler option, or queue feature is stable across versions.
- Determine whether scheduler and worker processes are configured and running only when environment access permits. Do not infer operational health from code configuration alone.
- Identify transaction boundaries, permissions, current credential storage, external service constraints, and any existing integration log or deduplication mechanism.

## 2. Classify the automation

Choose the smallest suitable mechanism:

| Mechanism | Suitable for |
|---|---|
| `doc_events` hook | Small synchronous lifecycle reaction or enqueueing work from a document event |
| Controller method | Behavior intrinsic to one app-owned DocType, especially validation and lifecycle rules |
| `scheduler_events` hook | App-defined recurring work at fixed supported intervals |
| Background job (`frappe.enqueue`) | Slow, bulk, or external work that should not block a web request |
| Whitelisted API method | An authenticated operation explicitly invoked over Frappe's API |
| Webhook endpoint | Receiving an external provider callback with provider-appropriate authentication/signature checks |
| Integration service | Reusable external request, mapping, and response handling |

Use built-in Frappe automation when it already fits. Do not add custom hooks or jobs merely because they are available.

## 3. Define behavior and operational needs

Capture:

- Trigger, target DocType, execution timing, and expected frequency.
- Inputs, outputs, side effects, and permissions/identity under which the work runs.
- Expected volume and duration, queue/timeout needs, and dependencies on workers or scheduler.
- Transaction timing: whether work must happen before commit, after commit, or can run later.
- Retryable versus permanent failures, idempotency key, duplicate handling, and recovery path.
- External authentication, timeouts, rate limits, payload constraints, logging, and alerting needs.

Do not invent business behavior or promise exactly-once execution. Clarify consequential decisions such as whether a failed external action blocks a document save.

## 4. Register hooks safely

- Edit the existing `hooks.py` and preserve its structure and mappings.
- Use fully qualified import paths that resolve to actual functions in the app.
- Verify the exact event signature and supported hook format for the installed version.
- Keep registered handlers short; delegate reusable or complex work to app modules.
- Avoid registering duplicate handlers for the same DocType/event unless multiple handlers are deliberate and ordering is understood.
- Treat install/migrate hooks as deployment lifecycle code: keep them idempotent and safe to rerun.
- Do not override core DocType classes or permissions when a narrower extension point suffices.

Typical `doc_events` shape:

```python
doc_events = {
    "Rental Booking": {
        "on_submit": "my_app.events.booking.on_submit",
    }
}
```

## 5. Design scheduled work

- Use only scheduler keys/frequencies supported by the installed Frappe version. Common options include `hourly`, `daily`, `weekly`, `monthly`, `all`, long-worker variants, and `cron`; verify exact availability and cron format before use.
- Keep a task bounded. Page or batch through large record sets rather than loading everything at once.
- Make tasks safe to run repeatedly and across multiple sites/workers. Use explicit time windows, stable cursors, or unique constraints as appropriate.
- Define timezone and boundary behavior for date-based schedules. Do not assume a scheduled task runs at an exact instant or exactly once.
- Distinguish a fixed app-defined schedule in `scheduler_events` from user-configurable schedules, which may use Frappe's scheduler records.
- Explain that changing scheduler hook configuration typically requires the version-appropriate migration step before the new schedule is active. Do not run migration or enable/disable the scheduler without explicit authorization.

## 6. Queue background jobs correctly

- Use `frappe.enqueue` (or a supported document enqueue API) for work that should not block the request. Select a queue and timeout based on expected duration and configured workers; do not assume custom queues exist.
- Pass serializable identifiers and small arguments, not open connections, request objects, or full document instances. Re-fetch and validate the document inside the worker.
- Check the document still exists and is in a valid state when the job runs; jobs may execute later than requested.
- Consider `enqueue_after_commit=True` or the installed version's supported after-commit mechanism when the job must only run after the initiating transaction commits. Enqueueing alone does not guarantee the document has committed.
- Avoid manual commit/rollback unless a specific, documented transaction boundary requires it. Let Frappe's job/request transaction behavior apply.
- Log useful context, then re-raise unexpected exceptions when the queue should record failure. Never silently swallow errors.
- Do not put credentials or sensitive data in job arguments or logs.

## 7. Build secure integrations and webhooks

For outbound APIs:

- Use HTTPS, explicit connect/read timeouts, bounded request sizes, and validate status codes and response structure.
- Follow the provider's authentication and rate-limit rules; honor `Retry-After` when appropriate.
- Keep credentials in an access-controlled Frappe Password field or approved secret store. Never hardcode, commit, expose, or log secrets.
- Validate configured destination URLs and restrict outbound hosts where appropriate to reduce SSRF risk.
- Avoid slow third-party requests inside document validation or save handlers unless synchronous completion is an explicit requirement.

For inbound webhooks:

- Treat guest access as unauthenticated transport, not proof of authenticity. Verify provider signatures using the documented algorithm and raw request body where required.
- Validate method, content type, payload size, schema, event type, and identifiers before processing.
- Add replay protection and durable deduplication by provider event ID. A simple “does this event exist?” check alone is not concurrency-safe; enforce uniqueness and handle races.
- Return a minimal provider-compatible response and appropriate status. Do not expose internal traces or sensitive data.
- Enqueue processing only after authentication/validation and ensure the accepted event is durably recorded before acknowledging it when delivery guarantees require that.

## 8. Retry and idempotency

- Separate transient failures (timeouts, connection resets, 429, selected 5xx) from permanent failures (invalid credentials, malformed payload, unsupported operation).
- Use bounded exponential backoff with jitter and provider `Retry-After` guidance. Set a finite attempt limit and define what happens after exhaustion.
- Do not blindly retry non-idempotent operations. Use provider idempotency keys or app-side operation/event records with a uniqueness constraint.
- Design for concurrent duplicate delivery, worker restarts, and a job that succeeds remotely but fails before recording local success.
- Persist enough status and error context for recovery when the business impact warrants it. Provide a safe manual retry/replay path where needed.
- Make state transitions explicit, for example Pending, Processing, Completed, Retry Scheduled, Failed, and Cancelled, while avoiding unnecessary tracking DocTypes for trivial work.

## 9. Logging and failure recovery

- Record operational identifiers: job/event ID, integration, related document, attempt, status, timestamps, and sanitized error category.
- Use Frappe logging and Error Log appropriately; create a dedicated integration log only when traceability/recovery needs justify it.
- Redact tokens, passwords, authorization headers, and sensitive payload fields. Avoid logging full customer or payment data by default.
- Document who can inspect and retry failures and how duplicate side effects are prevented.
- Do not add alerting infrastructure or a new log DocType without a clear operational need.

## 10. Keep the app architecture maintainable

Adapt to existing project conventions. A possible layout is:

```text
my_app/
  hooks.py
  events/
  tasks.py
  integrations/
  services/
```

- `hooks.py` registers entry points; event handlers coordinate lifecycle behavior; task modules expose scheduled/background entry points; integration modules handle external protocols; services own reusable business operations.
- Do not create empty layers or folders for a small integration. Keep app logic in source control rather than relying on undocumented site-only changes when reproducible deployment is required.
- Add dependencies only after confirming they are necessary and compatible with the app's supported Frappe/Python versions.

## 11. Validate and report

Review:

- Hook name, mapping shape, import path, handler signature, and duplicate registrations.
- Scheduler frequency/cron expression, migration implications, and idempotent bounded processing.
- Queue name, worker availability assumptions, timeout, argument serialization, and after-commit timing.
- Permission behavior and document state revalidation in asynchronous execution.
- Webhook authenticity, payload validation, replay/deduplication, and response behavior.
- API timeouts, credential storage, rate-limit handling, retry bounds, and response validation.
- Logging redaction and a practical recovery route.

Do not add or run tests unless the user asks to test or verify implementation. When asked, test the relevant behavior without sending real external requests unless authorized. Distinguish code review from tests actually run and from operational verification on a live bench.

Summarize the trigger, implementation files, scheduling/queue assumptions, transaction timing, security and retry strategy, and any required migration or site configuration. Keep deployment and production changes separate from code generation; do not run migrations, restart workers, enable schedulers, change production configuration, or send real external requests unless explicitly requested.

## Critical rules

- Always inspect the installed Frappe version and existing hook registrations.
- Never assume scheduler or workers are running without environment evidence.
- Never assume jobs execute exactly once or immediately.
- Always make repeatable external or scheduled operations idempotent where duplicate effects are possible.
- Never retry indefinitely or swallow unexpected exceptions silently.
- Never expose credentials or trust an unsigned webhook payload.
- Never assume enqueueing means the originating transaction has committed.
- Keep slow external work out of synchronous document events unless required.
- Never change production configuration or site data without authorization.
