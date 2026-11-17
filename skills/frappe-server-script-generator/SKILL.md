---
name: frappe-server-script-generator
description: Design, generate, review, and configure Frappe Server Scripts for document events, APIs, scheduled tasks, and other supported script types. Use when the user specifically wants a Frappe Server Script or needs a small server-side customization that fits the restricted Server Script runtime; prefer app code for maintainable or complex features.
---

# Generate a Frappe Server Script

Create the smallest safe Server Script that solves the user's requirement and matches the installed Frappe version. A Server Script is a database-configured Python script executed in Frappe's restricted runtime. It is not a normal Python module, and examples from ordinary Python or app controllers may not work in the sandbox.

## Inspect the project and runtime first

- Identify the bench, site, Frappe version, installed apps, target DocType, and existing scripts or hooks relevant to the request.
- Confirm the script type is supported by the installed version by checking version-matched docs, installed Server Script metadata/source, or existing configuration. Script types and events can vary by version.
- Determine whether the task belongs in a Server Script. Prefer app-owned hooks/controllers, whitelisted methods, or scheduled jobs for complex logic, reusable code, dependencies, robust testing, or normal source control. Use a Server Script when the user needs database-managed behavior and the requirements fit the sandbox.
- Check whether Server Scripts are enabled and whether the hosting environment allows them. Starting with Frappe v15, they are disabled by default; hosted shared benches may disallow them. Do not change bench or site configuration unless explicitly requested.
- Inspect the target DocType's fields, controller behavior, submit/cancel lifecycle, permissions, and neighboring scripts before writing code.

## Classify the request and choose the script type

Identify the trigger and configuration before generating code:

| Need | Common script type | Configuration to establish |
|---|---|---|
| Validate or update one document during its lifecycle | DocType Event | Reference DocType and exact event |
| Expose a callable endpoint | API | API method name, authentication, inputs, output, and rate limiting |
| Run work on a schedule | Scheduler Event | Supported schedule frequency and idempotency strategy |
| Filter records visible to a user | Permission Query | Reference DocType and safe query condition |
| Extend workflow behavior | Workflow Task or version-specific equivalent | Workflow integration and supported configuration |

Use only types and event names confirmed for the installed version. Document Event names commonly include before/after insert, validate, save, submit, cancel, delete, and print variants, but the exact choices must be checked against the installed framework.

## Specify behavior before writing code

For every request, identify:

- Trigger, target DocType, and event timing (or API/schedule/query trigger).
- Inputs and their source, including absent, malformed, and unauthorized input behavior.
- Required data changes, validation rules, side effects, and response shape.
- User identity and permission expectations; whether the operation must honor the caller's permissions or run with elevated privileges.
- Duplicate invocation, retries, recursion, and transaction behavior.
- Failure behavior and user-facing error messages.

Do not invent business rules, API access policy, or privileged behavior. Ask only when an ambiguity materially changes correctness or access control; otherwise state a limited assumption.

## Write for the restricted runtime

- Use only builtins and Frappe APIs exposed by the installed Server Script sandbox. Consult version-matched Server Script API documentation and installed `safe_exec` source when uncertain.
- Do not assume imports, arbitrary Python libraries, filesystem or network access, unrestricted `frappe` APIs, or normal Python globals are available.
- Keep code direct and small. Avoid abstractions that rely on unsupported language/runtime features.
- Validate all externally supplied values and check permissions before reading or changing records. Treat client-provided identifiers and fields as untrusted.
- Use permission-aware document APIs by default. Any use of `ignore_permissions`, direct database updates, or privileged reads must be necessary, narrowly scoped, and explicitly justified by the requirement.
- Do not expose secrets, private fields, stack traces, or unrestricted document data in API responses or logs.
- Avoid SQL where a supported document/query API suffices. If SQL is required, use parameterized values and enforce the intended permission and scope rules.
- Do not manually commit or roll back from an ordinary document event. Preserve Frappe's request transaction and lifecycle behavior.
- Avoid creating recursive document-event loops. If a script writes to the same or related DocType, reason about which events fire again and make the operation idempotent or guard it appropriately.
- Make API and scheduled scripts safe under repeated calls. Validate inputs, bound work, and apply rate limiting where supported and appropriate.

## Match logic to the lifecycle

- Use the earliest event that has the required validated data and the latest event consistent with the desired side effects.
- Use validation events to reject invalid documents; do not use after-save events for checks that must prevent persistence.
- Respect draft, submitted, and cancelled document behavior. Do not change submitted-document fields or bypass submit/cancel/amend rules.
- Keep workflow transitions in native Workflow configuration when possible. Do not reproduce workflow permissions or approval actions in a Server Script unless a specific missing rule requires server-side enforcement.
- For permission queries, return only the supported query condition format and avoid granting broader access than intended.

## Generate a reviewable result

Provide the script with its complete configuration:

1. Script type.
2. Reference DocType and event, API method, schedule, or query target as applicable.
3. Required enablement or rate-limit settings, without changing them unless requested.
4. Script body in a code block.
5. Explanation of inputs, permissions, side effects, and failure behavior.
6. Known version or sandbox assumptions.

For API scripts, also specify endpoint method name, HTTP method expectations if relevant, authentication/guest policy, accepted parameters, response shape, and rate limit. Guest access must remain off unless explicitly required and safe.

## Implement only in the right place

- When asked to implement, use the project's established storage/deployment method. Server Scripts are commonly records in the site's database rather than app source files; do not assume creating a Python file installs one.
- If the project uses fixtures or patches to manage Server Script records, follow that convention and keep the configuration reproducible.
- Do not activate or enable scripts, change global/site configuration, run migrations, install apps, or modify production data unless the user explicitly requested that operation.
- Keep code generation separate from saving the Server Script record, enabling the feature, migrating, and deploying it to a site.
- Do not add or run tests unless the user asks to test or verify implementation. When asked, use the project's available validation approach and do not claim sandbox behavior has been verified if it was only reasoned about.

## Review checklist

- Correct target, script type, event/API method/schedule, and version support.
- All fields and APIs exist and are spelled as in the installed project/version.
- Script syntax and constructs are compatible with RestrictedPython and the available safe globals.
- Inputs are validated; permissions and data exposure are appropriate.
- Lifecycle state and transaction behavior are respected.
- Writes are idempotent where retries or repeated events are possible.
- No accidental recursion, unbounded work, unsafe guest endpoint, or unnecessary privileged operation.
- Configuration and activation/deployment steps are clearly distinguished.

## Critical rules

- Never assume normal Python capabilities are available in Server Scripts.
- Never enable Server Scripts or guest access as an unrequested convenience.
- Never use a Server Script to bypass Frappe permissions or document lifecycle safeguards.
- Never present code as production-ready without identifying its trigger and required Server Script configuration.
- Always confirm the installed version's supported script types, events, and safe APIs before relying on them.
