---
name: frappe-client-script-generator
description: Design, generate, review, and configure Frappe Client Scripts and form scripts for field interactions, calculations, validation feedback, Link filters, child tables, buttons, and other Desk UI behavior. Use when implementing browser-side behavior for a Frappe form; not as a replacement for server-side validation, authorization, or business logic.
---

# Generate a Frappe Client Script

Create maintainable JavaScript form behavior that improves the user's Frappe Desk experience. Client-side code runs in the browser and can be bypassed; keep authoritative business rules and access control on the server. Follow the installed Frappe version, project conventions, and target DocType metadata.

## Inspect the project and form

- Identify the Frappe version, app, target DocType, relevant fields, child tables, and existing customizations.
- Inspect nearby Client Scripts, standard form scripts (`doctype.js`), Python controllers, Server Scripts, workflows, and whitelisted APIs to avoid duplication and locate the right layer.
- Confirm fieldnames and field types from DocType metadata; do not infer names from labels.
- Check which form events and APIs are supported by the installed version when uncertain.
- Decide whether the behavior is site-specific (Client Script record) or should ship across installations in app source (standard `{doctype}.js` form script). Follow the repository's established convention.

## Decide where the behavior belongs

Use a Client Script for form-specific user experience: dynamic visibility/mandatory/read-only state, field queries, immediate feedback, non-authoritative convenience calculations, or contextual buttons.

Use server-side code as well whenever the behavior affects correctness, security, data integrity, API/import/background operations, or records saved outside the standard browser form. Choose a controller, Server Script, permission rule, workflow, or whitelisted method according to the requirement. Do not put sensitive business operations exclusively in JavaScript.

## Translate the request into form behavior

Identify:

- Target DocType and user action.
- Triggering form event and why its timing is appropriate.
- Fields read or updated, including child DocType and table fieldname where relevant.
- Expected result, missing-value behavior, validation message, and document lifecycle restrictions.
- Any server-side validation or API dependency.
- User roles or document states that affect whether UI controls should appear.

Do not invent business rules such as date inclusivity, rounding, currency precision, or allowed state transitions. Confirm consequential ambiguity or state a narrow assumption.

## Select events deliberately

Common form events include `setup`, `before_load`, `onload`, `refresh`, `onload_post_render`, `validate`, `before_save`, `after_save`, `before_submit`, `on_submit`, `before_cancel`, and fieldname handlers. Exact events may vary by version; confirm before relying on less common events.

- Use `setup` for one-time form configuration, such as `frm.set_query`.
- Use `refresh` for UI state and contextual buttons. Make button creation idempotent across refreshes.
- Use a fieldname handler to react to that field changing.
- Use `validate` for browser-form feedback before save, while duplicating critical checks on the server.
- Use submit/cancel events only for UI behavior appropriate to that lifecycle point; never use them to bypass server validation.

Register handlers with the supported `frappe.ui.form.on` pattern and use the correct form or child handler arguments (`frm`, and `cdt`/`cdn` for child rows).

## Generate correct, focused JavaScript

- Use the exact DocType and fieldnames from metadata.
- Prefer documented `frm` APIs such as `frm.set_value`, `frm.toggle_display`, `frm.toggle_reqd`, `frm.set_df_property`, `frm.set_query`, and `frm.add_custom_button` when suitable and supported.
- Handle empty, zero, and null values explicitly; avoid truthiness checks that incorrectly treat valid zero values as missing.
- Avoid unnecessary server calls, duplicate handlers, hidden global state, and repeated work on every refresh.
- Make asynchronous flows explicit and handle errors. Reload the form only when server-side changes need to be fetched.
- For date and currency calculations, use framework utilities/formatting appropriate to the version and locale. Define inclusivity, timezone, precision, and rounding from the business requirement; do not silently assume them.
- For child tables, use the correct child DocType and row identifier, and update only the intended row using supported model APIs.
- Keep custom buttons visible only when relevant (saved state, document status, workflow state, and appropriate roles as UX hints). The server method must independently enforce permissions and allowed state changes.
- For Link filters, verify the linked DocType and filter fields. Treat client filtering as convenience; enforce access and business restrictions on the server.

## Security and data integrity

- Treat browser input, `frm.doc`, and request arguments as untrusted.
- Never include credentials, secrets, or privileged access assumptions in client code.
- Never treat hiding a field, disabling a button, a Link filter, or client-side role check as authorization.
- Never rely on client validation for a rule that must hold for API, import, background, or other non-form writes.
- Call server methods for sensitive operations; validate response and show a useful message without leaking internal errors.
- Do not write directly to the database from client code. Use supported form/model APIs and server endpoints.
- Respect submitted/cancelled document behavior and native workflow actions.

## Provide configuration and implementation

For each requested script, provide:

1. Target DocType and whether it is a Client Script record or app form script.
2. Event selection and rationale.
3. Field dependency map and any server-side dependency.
4. Complete JavaScript in a code block or the appropriate project file.
5. Configuration/setup steps, including required fields or API methods.
6. Assumptions and applicable permissions/lifecycle constraints.

If implementing in an app, place code according to the existing app structure and ensure its inclusion follows Frappe's conventions. If configuring a database Client Script, do not imply that a JavaScript file alone creates or activates that record.

## Review checklist

- Target DocType, fieldnames, child DocType, and event exist and match metadata.
- JavaScript syntax and Frappe APIs match the installed version.
- Field updates and calculations handle missing values, zero, precision, dates, and asynchronous behavior correctly.
- UI changes remain correct after refresh and for new, saved, submitted, and cancelled documents as applicable.
- Button and Link filter behavior is useful but not mistaken for server authorization.
- Critical validation and permission checks exist on the server.
- Existing form behavior is preserved and duplicate scripts are avoided.
- Configuration, server dependencies, and deployment steps are clear.

Do not add or run tests unless the user asks to test or verify implementation. When asked, use the project's existing verification approach and report what was actually checked.

## Critical rules

- Never generate a script without inspecting the target DocType and confirming its fields.
- Never reference a field that does not exist or confuse a label with a fieldname.
- Never confuse Client Scripts with Server Scripts or app form scripts.
- Never treat client-side validation or UI restrictions as security controls.
- Never implement sensitive or authoritative business logic exclusively in JavaScript.
- Always check installed-version support for events and APIs when uncertain.
- Keep scripts maintainable and avoid duplicating existing form behavior.
