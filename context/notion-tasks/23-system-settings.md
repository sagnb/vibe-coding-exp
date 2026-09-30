# 23 — Implement system settings and custom labels

## Source and usage

- Source: [Implement system settings and custom labels](https://app.notion.com/p/Implement-system-settings-and-custom-labels-3b44993602938052b4b5e9ed90017e43).
- Notion ID: `3b44993602938052b4b5e9ed90017e43`.
- Source domain: Administration & Configuration.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Implement a validated system-settings centre, safe localized help content, and editable English/French custom labels for projects. Keep global configuration in the Control DB and project/protocol configuration in the project's isolated DB. Do not migrate the legacy ReLiS Editor/Tomcat settings: the agreed ReLiS2.0 architecture uses native protocol forms and has no DSL/editor import pipeline.
- **Functional Domain:** Administration & Configuration
- **Behavior:**
  - `config_admin_configuration.php` defines one legacy admin configuration record with editor URL, editor workspace path, debug-comment flag, first-connect flag, and paper-title trim length. Its UI exposes editor location/path and debug-comment state.
  - `config_configuration.php` defines an extensive project configuration record, grouped in the UI as Papers, Screening, Quality Assessment, Classification, and DSL. It includes import/export CSV separators, paper-key settings, workflow toggles, reviewer/validation settings, QA cut-off, and extraction-validation settings.
  - Legacy project menus in `Manager_lib.php` read configuration flags to show or hide workflow modules and actions.
  - Legacy `lng`/`lng_min` helpers resolve labels through language management and support an edition mode; project/entity configuration contains user-facing labels for criteria, questions, fields, and operations.
  - The legacy admin and project configuration include an editor URL/workspace because they integrate with the external ReLiS Editor/Tomcat workflow. This has been explicitly removed from the ReLiS2.0 migration scope.
  - Legacy Help/Home content exists as application views/pages, but it is not a modern versioned, localized content-management workflow.
- **Users and permissions:**
  - Application Administrator manages global system settings, static help content, global label defaults, and feature availability. All changes are audited.
  - Project Manager cannot modify global settings; a project-specific configuration right must be granted explicitly if needed.
  - Standard User can set only their own language preference and has no settings-management access.
  - Project roles: Project Admin manages project settings/protocol labels within their own project only, including localized labels for screening criteria, QA questions, and extraction fields. Reviewer and Validator can see the effective localized labels but cannot edit them. Guest has no settings or label-editing access.
- **UI behavior:**
  - Provide an Application settings area for Application Administrators, grouped by safe categories such as defaults, email/notifications, display, storage/retention status, and feature availability. Each setting includes a description, valid range/format, effective value, and change history.
  - Provide a Project settings entry point that links to the native Project & Protocol Management forms ([21](21-project-settings.md)) rather than duplicating those configurations in a global admin screen.
  - Provide a Custom labels editor for Project Admins, showing English and French values, fallback/default label, context/field key, preview, validation errors, and revision history.
  - Provide a localized Help content editor with constrained rich text or Markdown, preview, draft/published state, and rollback. Render published help content safely without executable HTML/JavaScript.
  - Do not show ReLiS Editor URL, workspace path, DSL, Xtext, Tomcat, or "import from editor" controls.
- **API behavior:** Implement stable use cases: `getSystemSettings`; `validateSystemSettingsDraft`; `updateSystemSettings`; `getProjectEffectiveSettings`; `listCustomLabels`; `updateCustomLabelDraft`; `publishCustomLabelRevision`; `getHelpContent`; `saveHelpContentDraft`; `publishHelpContent`; `restoreSettingsOrContentRevision`. The settings registry defines type, validation, sensitivity, scope, default, restart requirement, and safe rollback strategy. The API must reject unknown keys, secrets in responses, and attempts to edit a scope outside the caller's authority.
- **Data model:**
  - Data lives in Both databases. The Control DB holds typed global settings, immutable setting revisions, global translation defaults, help-content drafts/publications, and audit events. Each Project DB holds project/protocol settings and project-specific label overrides linked to stable protocol field/criterion/question IDs.
  - Store localized values by locale and stable key, never by translated display text alone. An effective-label resolver applies project override, global default, then static UI translation fallback.
  - Runtime secrets remain environment-managed and are not stored in the settings UI/database. The Settings UI may show non-sensitive capability/status information only.
- **Acceptance criteria:**
  - [ ] An Application Administrator can edit only registered, validated global settings and can see effective value, description, history, and safe rollback availability.
  - [ ] A Project Admin can edit only project-scoped protocol settings and localized custom labels in their own isolated project database.
  - [ ] The effective settings resolver applies project value, then system default, and never exposes another project's values.
  - [ ] Feature toggles affect menus, UI states, and API authorization consistently; disabled capabilities cannot be reached by direct URL/API call.
  - [ ] Custom labels support English and French, use stable IDs, provide fallback values, and remain traceable to a project/protocol revision.
  - [ ] Help content is localized, versioned, sanitized, previewable, and publishable only by an Application Administrator.
  - [ ] No ReLiS2.0 UI, API, or setting persists or relies on legacy editor/Tomcat/DSL configuration.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Manage validated global settings, English/French labels, and safe localized help.

## Proposed prerequisites

- [15 — Implement Project Context, Roles, and Membership](15-membership.md)
- [21 — Implement Direct Project Settings Management](21-project-settings.md)
- [22 — Implement audit and undo UI](22-audit-undo.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Define a settings registry with type, scope, default, validation, sensitivity, restart requirement, and safe rollback strategy.
2. Implement `getSystemSettings`, `validateSystemSettingsDraft`, `updateSystemSettings`, and `getProjectEffectiveSettings`; reject unknown keys, secrets, and unauthorized scopes.
3. Separate global settings/revisions in the Control DB from protocol settings/labels in the Project DB.
4. Implement `listCustomLabels`, `updateCustomLabelDraft`, and `publishCustomLabelRevision` using stable IDs and project → global → static translation fallback.
5. Implement `getHelpContent`, `saveHelpContentDraft`, `publishHelpContent`, and `restoreSettingsOrContentRevision` with drafts, preview, sanitization, and history.
6. Create settings, labels, and help screens by role; project-configuration links reuse the dedicated module ([21](21-project-settings.md)).

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/Data model fields.

- [ ] `updateSystemSettings` only accepts keys present in the settings registry; unknown keys are rejected, secrets are never included in any settings-read response, and a caller outside a setting's declared scope is rejected server-side.
- [ ] Each setting's effective value, description, valid range/format, and change history are visible to Application Administrators in the Application settings UI, with a safe rollback path (`restoreSettingsOrContentRevision`) for any previous revision.
- [ ] `getProjectEffectiveSettings` resolves project override → global default → static UI fallback, in that order, and never returns another project's override value when called for project A.
- [ ] A feature toggle that hides a menu/UI state also blocks the corresponding API route for a direct call — verified by hitting a disabled-capability endpoint directly and getting a rejection, not a filtered/empty success.
- [ ] Custom labels are keyed by a stable ID (never by translated display text), store both English and French values plus a fallback/default, and are editable only by a Project Admin within their own project — Reviewers/Validators see the effective label read-only, Guests have no access.
- [ ] Publishing a custom-label revision keeps prior revisions retrievable/traceable to the project/protocol state they applied to.
- [ ] Help content supports draft/published states, is rendered with sanitization that strips executable HTML/JavaScript (verify with a test injecting a `<script>` payload into draft content), and can only be published by an Application Administrator.
- [ ] The Project settings entry point in this feature links out to [21 — Direct Project Settings Management](21-project-settings.md) rather than re-implementing protocol configuration here.
- [ ] No screen, endpoint, or stored setting references the legacy ReLiS Editor URL/workspace path, DSL, Xtext, or Tomcat — these are explicitly out of scope for ReLiS2.0.
- [ ] Automated tests cover: an unknown-key update rejection; a Project Admin editing labels only in their own project; the fallback-resolution order for effective settings; help-content sanitization against injected markup; and a non-administrator rejected from publishing help content.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

The labels/global-settings revision model must not reintroduce protocol revisions prohibited in [21 — Direct Project Settings Management](21-project-settings.md). Project Admin versus Owner/Manager permissions must be reconciled per [pending decisions](PENDING-DECISIONS.md) item 2.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [23.01 — Define a settings registry with type, scope, default, validation, sensitivity, restart requirement, and…](sub-issues/23-01.md)
- [23.02 — Implement getSystemSettings, validateSystemSettingsDraft, updateSystemSettings, and…](sub-issues/23-02.md)
- [23.03 — Separate global settings/revisions in the Control DB from protocol settings/labels in the Project DB](sub-issues/23-03.md)
- [23.04 — Implement listCustomLabels, updateCustomLabelDraft, and publishCustomLabelRevision using stable IDs and…](sub-issues/23-04.md)
- [23.05 — Implement getHelpContent, saveHelpContentDraft, publishHelpContent, and restoreSettingsOrContentRevision…](sub-issues/23-05.md)
- [23.06 — Create settings, labels, and help screens by role](sub-issues/23-06.md)
