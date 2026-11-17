---
name: frappe-app-creation
description: Create or extend applications for the Frappe Framework, from app scaffolding and DocTypes to server logic, permissions, UI, and integration. Use when implementing a Frappe app or feature in a Frappe codebase; not for general Python or ERPNext usage questions.
---

# Create a Frappe app

Implement the user's requested app or feature using the conventions of the installed Frappe version and the current repository. Prefer a working, maintainable vertical slice over scaffolding unrelated features.

## Establish the project context

- Inspect the repository before editing. Determine whether this is an existing Frappe app, a bench, or a non-Frappe project; locate app metadata, DocTypes, hooks, fixtures, patches, and existing tests.
- Identify the Frappe version and other installed apps from the project or bench. Use documentation that matches that version; if version-specific behavior is unclear, consult the official Frappe Framework docs rather than guessing.
- If no bench or app exists, explain the expected location and use the Bench workflow only when the environment supports it. Creating an app (`bench new-app`) and installing it on a site (`bench --site <site> install-app`) are separate operations.
- Clarify requirements only when a consequential product decision cannot be inferred. Otherwise choose a small, conventional implementation and state material assumptions.

## Design and implement

- Turn the request into the smallest complete user workflow. Identify its data, roles, lifecycle, validation rules, and user-facing entry point before adding files.
- Use Frappe's native structures: DocTypes and their controllers for persisted records and validation; `hooks.py` for framework integration; pages, forms, reports, or website routes for the UI the request needs; patches for data or schema changes that cannot be represented declaratively.
- Follow neighboring app conventions for module names, Python style, JavaScript, translations, fixtures, and asset organization. Do not assume ERPNext is installed or add it as a dependency unless requested or required by inspected code.
- Enforce access through DocType permissions and role checks. Validate data on the server even when the client also validates it. Avoid exposing private fields or relying on client-side checks for authorization.
- Keep changes focused. Reuse existing framework APIs and app abstractions; do not duplicate core functionality or introduce dependencies without a clear need.
- Preserve existing data and behavior. Treat uninstall, reinstall, reset, destructive schema changes, and site-wide changes as separate high-impact operations; do not perform them as part of ordinary app implementation.

## Finish the requested workflow

- Ensure metadata, module paths, imports, hooks, and asset references agree with the app's actual structure.
- Include migration-safe schema or data changes when needed. Do not assume that creating files installs the app or updates a site.
- Describe the files and user workflow delivered, commands needed for the user's bench/site, and any setup or assumptions. Keep install, migration, build, and deployment steps distinct and appropriate to the environment.
- Use official references as needed: [Create an App](https://docs.frappe.io/framework/user/en/tutorial/create-an-app), [Apps](https://docs.frappe.io/framework/user/en/basics/apps), and the documentation for the installed Frappe version.
