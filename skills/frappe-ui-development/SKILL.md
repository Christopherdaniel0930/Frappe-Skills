---
name: frappe-ui-development
description: Design and implement usable, accessible, responsive interfaces for Frappe applications, including forms, lists, workspaces, dashboards, reports, portals, and custom pages. Use when a request involves Frappe UI/UX design or frontend implementation; not for backend-only changes.
---

# Frappe UI Design and Development

Design and implement interfaces around real user tasks and real application capabilities. Prefer native Frappe views and components when they fit. Add custom UI only when it supports a clear requirement that standard functionality cannot meet. Follow the installed Frappe version and the existing app's visual and technical conventions.

## 1. Understand the project and users

- Inspect the repository, Frappe version, installed apps, existing theme and components, target DocTypes, fields, workflows, permissions, pages, workspaces, and scripts relevant to the request.
- Identify target users, their common tasks, information needs, decision points, and access levels.
- Map the main user journey: entry point, action, required information, system response, success state, error/empty state, and next action.
- Check for existing UI that already provides the requested function. Avoid duplicate navigation, forms, dashboards, or actions.
- Confirm which UI surface is intended: Desk form/list, Workspace, report, Web Page, Portal, dialog, or custom Page. If unclear, choose the smallest native surface that meets the need and state the assumption.

## 2. Choose the right Frappe surface

| Surface | Use when |
|---|---|
| DocType Form | Users create, review, or update one document |
| List View | Users browse, filter, sort, and act on a set of documents |
| Workspace | Users need module navigation, shortcuts, and summaries |
| Report | Users need structured, filterable analysis of records |
| Dashboard/Chart | Users need a small set of meaningful metrics or trends |
| Web Page or Portal | Users need a website-facing or self-service experience |
| Custom Page | A specialized multi-step workflow cannot be expressed well with native views |
| Dialog | A focused, brief action that does not warrant a full page |

Prefer native DocType, Workspace, List, Report, and dashboard features before custom HTML/CSS/JavaScript. Use custom Pages when the interaction genuinely needs them, not merely to restyle a standard form.

## 3. Define content and interaction before visuals

For each page or view, specify:

- Pagnneeds.

Every component must have a user purpose. Do not create visual elements to make a page appear more elaborate.

## 4. Use a coherent visual system

- Follow the installed Frappe version's visual language, the app's existing theme, and established component patterns.
- Establish a clear hierarchy with concise headings, grouped content, readable labels, and prominent primary actions.
- Use consistent spacing, alignment, typography, and button treatment. Keep color restrained and meaningful; do not communicate status through color alone.
- Use cards, badges, icons, borders, and animation only when they improve scanning, meaning, or interaction.
- Avoid excessive cards, gradients, oversized headings, decorative icons, arbitrary charts, redundant badges, repeated information, unnecessary modals, and generic landing-page patterns.
- Keep CSS scoped to the app/page/component. Avoid broad selectors or overrides that can break Desk or other apps.
- Reuse established Frappe and app components; do not add a frontend dependency for a small UI detail.

## 5. Design forms, lists, reports, and dashboards

### Forms

- Verify every field against the target DocType metadata.
- Group fields into meaningful sections and columns; prioritize fields users need to complete the task.
- Place child tables with related information and avoid needless scrolling or duplicate fields.
- Respect required/read-only behavior, workflow state, and submitted/cancelled lifecycle.
- Use concise descriptions for fields that need explanation; do not use help text to restate the label.

### Lists and reports

- Show identifying and decision-relevant columns without overwhelming the view.
- Provide useful filters, sorting, grouping, and totals for the actual workflow.
- Make status and row actions understandable and consistent with permissions.
- Ensure report definitions and displayed totals reflect the underlying records and filter scope.

### Dashboards and workspaces

- Use real, permission-appropriate data and label the metric's period and meaning.
- Choose KPIs, queues, trends, or shortcuts that help the user decide or act.
- Link summaries to the relevant filtered records when useful.
- Never invent sample metrics or imply live data when using placeholders. Clearly label design mock data.

## 6. Implement interactions safely

- Connect every button and navigation item to a real action. Do not leave placeholder controls in a completed implementation.
- Use Client Scripts for browser-only convenience and feedback. Enforce permissions, validation, and important business rules on the server.
- For sensitive actions, call a permission-checked server method and handle success and failure clearly. A hidden or disabled button is not authorization.
- Respect workflow actions and document lifecycle; do not directly manipulate status to bypass server behavior.
- Keep JavaScript maintainable, avoid duplicate handlers, and follow app conventions for standard form scripts versus database Client Scripts.
- Preserve existing behavior and avoid modifying Frappe core files.

## 7. Accessibility and responsive behavior

- Keep text readable and contrast sufficient. Do not rely on color alone for status or meaning.
- Use meaningful labels, descriptions, and button names; make interactive controls understandable to keyboard and assistive technology users where the chosen framework supports it.
- Preserve logical focus order and visible focus states. Use semantic structure for custom pages.
- Ensure common screen widths remain usable: prevent unintended horizontal overflow, keep key actions accessible, and adapt dense tables and layouts to the installed framework's responsive patterns.
- Do not add motion that distracts or blocks use; respect reduced-motion preferences when adding animation.

## 8. Plan and implement

For a design request, provide a concise implementation plan that identifies:

1. Target users and primary journey.
2. Chosen Frappe surface and why it fits.
3. Layout and component structure.
4. Data sources, fields, and actions.
5. Permission and workflow constraints.
6. Responsive and accessibility behavior.

When implementation is requested:

- Inspect neighboring code and follow app structure for pages, assets, scripts, styles, and translations.
- Keep changes focused; reuse native features and existing components.
- Use actual project data and functioning actions.
- Keep site setup, mi   2gration, and deployment separate from code changes. Do not change production data or deploy unless requested.
- Do not add or run tests unless the user asks to test or verify implementation. If asked, report what was actually checked and any limitations.

## 9. Review the result

Review the implemented interface for:

- Functional behavior: navigation, actions, form fields, filters, and data display.
- Visual consistency: hierarchy, spacing, typography, and restrained use of color.
- Usability: primary action clarity, concise forms, helpful error and empty states, and minimal unnecessary steps.
- Accessibility and responsive layout at relevant widths.
- Technical fit: installed Frappe version, app conventions, permissions, no broken assets, and no broad CSS conflicts.

Do not claim browser, responsive, or accessibility verification unless it was actually performed.

## Critical rules

- Function and user needs come before decoration.
- Never invent business data or display fake metrics as real.
- Never include a component without a clear purpose.
- Prefer standard Frappe functionality over unnecessary custom UI.
- Never treat client-side controls as security enforcement.
- Never leave placeholder interactions in a completed implementation.
- Always inspect the existing application before redesigning or extending its UI.
