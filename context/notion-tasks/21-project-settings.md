# 21 — Implement Direct Project Settings Management

## Source and usage

- Source: [Implement Direct Project Settings Management](https://app.notion.com/p/Implement-Direct-Project-Settings-Management-3b8499360293808f93eed7d8bda08f06).
- Notion ID: `3b8499360293808f93eed7d8bda08f06`.
- Source domain: Project & Protocol Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist. This is one of the longest and most internally conflicted source cards — read [pending decisions](PENDING-DECISIONS.md) item 11 before implementing.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Provide a native Project workspace > Settings area where authorized users directly manage the current project settings.
- **Functional Domain:** Project & Protocol Management
- **Behavior:** Legacy ReLiS lets Project Admin directly edit project settings, Paper Sources, Search Strategies, Screening phases, exclusion criteria, Quality Assessment settings, extraction configuration, and reporting configuration. NeoReLiS preserves these concepts, but uses fixed Prisma models and normal CRUD instead of generated SQL, dynamic tables, views, procedures, or editor-generated configuration.
- **Users and permissions:**
  - Only `PROJECT_OWNER` can access `/settings` in their own project.
  - `PROJECT_OWNER` can add, edit, and delete active protocol settings.
  - Application Administrators have audited global access to project protocol settings.
  - `REVIEWER`, `VALIDATOR`, and `GUEST` users cannot access or modify Protocol settings.
  - `PROJECT_OWNER` remains a protected backend-managed role; this ticket must not allow it to be assigned, changed, or removed.
- **UI behavior:**
  - Settings entry in project workspace navigation, using the same high-level structure as the Project Creation Wizard, as tabs: General settings; Screening; Quality assessment; Data extraction; Reporting.
  - **Access rules** — Project Owner and Application Administrator access every tab; Project Manager can access/manage Screening, Quality assessment, Data extraction, Reporting but must **not** access or modify General settings; Reviewer/Validator/Guest cannot access Settings at all. Backend authorization remains mandatory even when a tab/action is hidden.
  - **Direct editing pattern** — use direct editing rather than add/edit dialogs: Add creates a new editable row; Edit turns the row into editable controls (or expands below it for complex config); Save validates and immediately persists; Cancel restores previous values or removes an unsaved new row; Delete uses inline confirmation (Cancel/Delete). Show field-level validation errors next to the control. Show a clear success message after save and a clear integrity error when a change cannot be safely applied. Do not save incomplete/invalid rows. Do not use dialogs except for the Screening phase configuration pop-up.
  - **General settings** (Owner/Admin only) — Export project configuration button (downloads current config as JSON). Project details: title, short name (not editable), description — editable. Research questions table (Question | Description | Order | Actions) with move-up/move-down instead of a typed order value.
  - **Paper sources** (when Paper Library & Import enabled) — table Source name | Description | Collection method | Original search query | Actions; source name required+unique; collection method required (Automatic/Manual); a source already used by an import/paper is frozen (cannot edit/delete, may be archived).
  - **Search strategies** — table Strategy name | Description | Actions; name required+unique; frozen once used, archivable. When Paper Library & Import is disabled, show an empty state + an "Enable" action; confirming enables the module immediately.
  - **Screening** (when enabled) — workflow rules section (reviewers per paper, conflict type, conflict resolution, "assign to validators who haven't screened", use kappa, validation percentage, validation outcome handling, screening inclusion mode) shown inline-editable; validation: reviewers per paper required >0; conflict type/resolution required when reviewers>1; validation fields available only when Screening Validation is enabled; show "Not configured" for unset optional values.
  - **Screening phases** — table Order | Title | Description | Displayed fields | Configuration type | Final phase | Actions. A phase has title (required), description (optional), paper source (All papers/Previous phase), source paper status (All/Included/Excluded), one+ displayed fields (Title/Abstract/Preview/BibTeX/Link), configuration type (Default/Customised), final phase (Yes/No, unique per project). Backend manages order; UI offers move-up/move-down only, never a manual order field. A phase using "Previous phase" depends on server-managed order; reordering is rejected if it would invalidate started work. Only one active final phase per project.
  - **Phase configuration pop-up** (the one allowed dialog) — clicking the Configuration type cell opens it. For Default: show inherited general settings, offer "Customise this phase" (creates a phase-specific config initialized from current general settings). For Customised: editable phase-specific settings (same validation rules as general), offer "Use default configuration" (removes the override, reverts to current general settings) — available even after the phase has started/has assignments/decisions/validation work. Before saving, show a clear impact warning explaining which settings change and that existing assignments/decisions are preserved without automatic deletion/reset/reassignment/reinterpretation.
  - **Exclusion criteria** / **Inclusion criteria** — table Criterion | Description | Actions; direct add/edit/inline-confirmed delete. At least one screening phase is required while Screening is enabled. When Screening is disabled, show empty state + enable action.
  - **Quality assessment** (when enabled) — rules section (minimum total score threshold, validation enabled, default validation percentage) inline-edit, "Not configured" for unset. Questions table (Question | Actions). Answers table (Answer | Score | Actions) — an answer requires label+score. When disabled, empty state + enable action.
  - **Data extraction** (when enabled) — Extraction categories table: Category name | Label | Type | Required | Configuration | Depends on | Subcategories | Actions. Adding creates an editable row; editing/adding a subcategory/complex config expands an inline area below the row. A category has: type (Simple/List/DynamicList/Dependent DynamicList, required); name (required, unique per project); label (optional); required/optional; optional subcategories via "+ Add subcategory" (nested editable row).
    - Simple: value type (Boolean/Integer/Real/Date/String/Text); optional validation pattern (RegEx); optional max text length (prefilled 20 for Text, editable/removable); optional default value with input adapted to the value type.
    - List: list options (add multiple; at least two distinct required); optional max selectable values (unlimited if unset).
    - DynamicList: optional initial values (add multiple); optional max selectable values.
    - Dependent DynamicList: source DynamicList category — required, dropdown of eligible existing DynamicList categories; optional max selectable values.
    - When disabled, empty state + enable action.
  - **Reporting** (when enabled) — standard project statistics generated from workflow data; no configuration controls when nothing is configurable. Custom Chart Reports (when enabled): table Report name | Dimension mode | Category/X-axis | Y-axis | Chart type | Actions; add/edit via inline expandable row. Dimension mode Simple/Compare; chart type Line/Pie/Bar/Bubble chart (for Compare). Simple: one extraction category + one-or-more chart types. Compare: unique report name; X-axis category; Y-axis category; multiple chart types. Only compatible active Data Extraction categories selectable. When Reporting/Custom Chart Reports disabled, empty state + enable action for the required module.
  - **Audit metadata and hidden columns** — every persisted settings record stores backend-managed `Created by`, `Created at`, `Modified by`; where displayed in a table, these are hidden columns by default behind a "Show hidden columns" control usable only by Project Owner and Application Administrator (Project Managers cannot reveal them); these values are read-only and backend-authoritative; the client never submits/modifies them.
- **API behavior:**
  - Hono only as HTTP/API layer: validates commands, calls application services, returns responses; must not generate Prisma schemas/migrations/SQL/tables/columns/views/procedures/per-category structures from settings input.
  - Reuse the relevant Zod schemas and structured controls from the Project Creation Wizard ([18](18-protocol-wizard.md)).
  - Do not create or use protocol-revision drafts, protocol-version comparison, activation, rollback, or provisioning endpoints in this ticket.
  - **Authorization** — General settings endpoints: Project Owner + Application Administrator only. Screening/QA/Data Extraction/Reporting endpoints: Project Owner, Project Managers, Application Administrator. Reviewers/Validators/Guests: no access. Project Managers must not retrieve/modify project metadata, Research Questions, Paper Sources, or Search Strategies. Every endpoint enforces URL project ID, project state, membership, role, and module-enabled rules.
  - **Settings use cases** — retrieve current settings per tab; export current project configuration as JSON (Project Owner only, data from Control DB); add/edit/delete/reorder Research Questions; add/edit/archive/delete Paper Sources and Search Strategies; update Screening workflow rules; add/edit/delete/reorder Screening phases; retrieve/update a phase-specific Screening configuration; switch a phase between Default/Customised; add/edit/delete Inclusion/Exclusion criteria; update QA rules; add/edit/delete QA questions and response options; add/edit/delete Data Extraction categories/options/dependencies/subcategories; add/edit/delete Custom Chart Reports; enable/disable supported modules when authorization/integrity rules allow.
  - **Phase configuration behavior** — a Default phase resolves from current general Screening config; switching Default→Customised initializes a phase-specific config from current general settings; updating a Customised phase changes only that phase; switching back to Default removes/deactivates the override. Backend manages phase order and accepts an explicit reorder command updating all affected positions in one transaction. Client must not submit `createdBy`, `createdAt`, or a manually chosen order value.
  - **Audit metadata** — every created settings record stores backend-managed `createdBy`/`createdAt`, read-only; read models include audit columns only for Owner/Admin; Project Managers must not receive/reveal them; every mutation also writes an audit event.
  - **Mutation behavior** — for every mutation: authorize the caller; validate with Zod; perform normal Prisma CRUD against the existing fixed-schema table(s); use a transaction when one action touches multiple related rows; return the updated setting or field-level validation errors; return impact warnings when confirmation is required; write an audit event.
  - **Direct-update integrity rules** — a Paper Source/Search Strategy referenced by imported papers cannot be physically deleted/edited (may be archived); a Screening Phase with assignments/decisions/completed work cannot be deleted; a criterion used by a screening decision cannot be deleted; a QA question/response option used by completed assessments cannot be deleted/structurally changed; a Data Extraction category/option/dependency/field-type/subcategory used by extracted data cannot be deleted/structurally changed; only one active final Screening phase may exist; reordering phases is rejected when it would make an existing "Previous phase" workflow inconsistent; a module with active runtime data cannot be disabled if doing so would make the data inaccessible or invalidate the workflow. Changes to reviewer count, conflict rules, validation settings, or a phase configuration remain allowed after a phase has started — but require a clear impact warning + explicit confirmation first, and must preserve existing work (no automatic delete/reset/reassign/reinterpret). Only invalid values or impossible integrity conditions block the save. Return clear, human-readable validation errors/impact warnings — never internal database/Prisma/infrastructure errors.
- **Data model:**
  - Control DB — Project metadata; Research Questions and their display order; audit events for Control DB mutations.
  - Project DB (fixed Prisma models) — enabled module settings; Paper Sources; Search Strategies; Screening workflow rules; Screening phases; Inclusion/Exclusion criteria; Screening Validation settings; QA rules/questions/response options/scoring/thresholds; Data Extraction categories/options/dependencies/multi-value rules/subcategories; Reporting and Custom Chart Report definitions; project-scoped audit events.
- **Acceptance criteria** (this Notion property, as consulted, restricts the workspace's navigation label and directly contradicts several UI/API behavior statements above — see Gaps below): the workspace contains "Project administration > Protocol settings" navigation; Protocol settings separated into Review scope & discovery, Screening, Quality assessment, Data extraction, and Reporting pages; no protocol-revision draft/version-comparison/activation/rollback/change-impact UI is present; clicking Save in a settings **dialog** immediately persists the validated change (note: contradicts the "no dialogs except phase pop-up" UI-behavior rule above); refreshing displays the saved active configuration; Research Questions can be added/edited/ordered/deleted; Paper Sources support name/description/method/optional search query, used only as screening keyword-highlighting metadata (never executed as an external search); Search Strategies are independent named records with optional descriptions; Screening supports workflow rules/phases/exclusion/inclusion criteria; QA supports rules/questions/response options/scoring/validation settings; Data Extraction supports the category types/validation rules from the Wizard; Reporting supports eligible Custom Chart Report definitions; unsafe destructive changes are rejected with clear errors; all configuration updates use fixed-schema Prisma CRUD and never generate database structure; all mutations are authorized and audited.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Perform potentially expensive approved migration or consistency work through an idempotent revision job. Persist validation, impact, progress, and failure reports for the revision. A failed job must not activate the revision or leave partially applied configuration visible to users. Re-running the same activation request must not create duplicate protocol versions or duplicate migration work. *(This directly contradicts the Description/UI/API behavior's "no revision workflow, no protocol-version activation" — see Gaps.)*
- **Audit:** Empty

## Objective and summarized requirements

Directly edit active settings while preserving the integrity of existing work.

## Proposed prerequisites

- [18 — Build the Native Project Protocol Wizard](18-protocol-wizard.md)
- [20 — Implement Project Workspace Navigation and Home Dashboard](20-project-workspace.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Reuse wizard contracts/controls and organize General, Screening, QA, Data extraction, and Reporting.
2. Apply the detailed UI/API matrix: General for Owner/Admin; other tabs for Owner/Manager/Admin, after resolving the permissions-summary conflict.
3. Implement inline Add/Edit/Save/Cancel/confirmed Delete; only phase configuration uses a popup. Short name is read-only; question/phase reordering is backend-managed.
4. Manage questions, sources/strategies, and JSON export. Used sources/strategies are frozen and may only be archived where permitted.
5. Manage screening defaults, criteria, and phases; allow one final phase, transactional ordering, and a consistent Previous phase flow. Custom copies defaults; switching back removes the override.
6. Allow rule changes after work starts with an impact warning and confirmation, preserving decisions/assignments without automatically reinterpreting previous work.
7. Manage QA, categories/subcategories/lists/dependencies, and charts through fixed-schema CRUD. Block deletion/structural changes of used items and module disabling that invalidates data.
8. Keep `createdBy`/`createdAt`/`modifiedBy` backend-managed; hidden audit columns are Owner/Admin-only and must not leak to Managers.
9. Audit mutations and return field errors; do not generate schemas or implement a revision workflow without resolving the source contradiction.

## Definition of Done — completion checklist

This card's own Acceptance criteria, UI behavior, and Jobs behavior properties directly contradict each other (dialogs vs. no dialogs; direct persistence vs. a revision/activation job) — see Gaps. The checklist below states what is safe to build now (the direct-editing core, consistent across Description/UI/API/Data model) and flags what must not be built without a resolved decision.

- [ ] `/settings` (or the equivalent workspace route) is reachable only by the Project Owner and Application Administrator for **General settings**, and additionally by Project Managers for **Screening/QA/Data extraction/Reporting** — verified server-side per endpoint, not only via hidden tabs.
- [ ] Every settings table (Research Questions, Paper Sources, Search Strategies, Screening phases, criteria, QA questions/answers, Data Extraction categories, Custom Chart Reports) uses **direct inline editing** (Add → editable row, Edit → inline controls, Save → immediate persist + validation, Cancel → revert/remove, Delete → inline confirm) — the only pop-up dialog anywhere in this feature is the Screening phase configuration pop-up.
- [ ] Project short name remains read-only in General settings; Research Question and Screening phase ordering use move-up/move-down actions only — no manually editable numeric order field is exposed, and the backend manages the actual order value and reorders atomically.
- [ ] A Paper Source or Search Strategy already referenced by an import/paper cannot be edited or deleted — only archived (when the caller is authorised).
- [ ] JSON configuration export is available only to the Project Owner and reflects Control DB + Project DB protocol data without duplicating or reintroducing a protocol-revision concept.
- [ ] The Screening phase configuration pop-up: shows inherited settings for a Default phase with a "Customise this phase" action; shows editable phase-specific settings for a Customised phase with a "Use default configuration" action (available even after the phase has assignments/decisions); and — for any change to reviewer count, conflict rules, validation settings, or phase configuration — displays a clear impact warning and requires explicit confirmation before saving, without deleting/resetting/reassigning/reinterpreting existing assignments or decisions.
- [ ] Only one active final Screening phase exists per project at a time; reordering phases is rejected when it would make an existing "Previous phase" configuration inconsistent with the new order.
- [ ] A Screening Phase with assignments/decisions/completed work, a criterion used by a decision, a QA question/option used by a completed assessment, or a Data Extraction category/option/dependency/subcategory used by extracted data cannot be deleted or structurally changed — attempting to do so returns a clear, human-readable integrity error (never an internal DB/Prisma error).
- [ ] A module cannot be disabled while it has active runtime data that would become inaccessible or invalidate the workflow.
- [ ] `createdBy`, `createdAt`, and `modifiedBy` are always backend-set (never accepted from the client); where shown, these audit columns are hidden by default behind a "Show hidden columns" control usable only by the Project Owner and Application Administrator — a Project Manager cannot reveal them even by direct API inspection.
- [ ] Every mutation in this feature performs normal Prisma CRUD against the already-fixed schema from [04](04-project-prisma.md) — it never generates a table, column, view, procedure, migration, or per-category structure — and writes an audit event.
- [ ] Automated tests cover: an Owner-only General-settings edit rejected for a Project Manager; a frozen (used) Paper Source rejecting edit/delete; a Screening-phase reorder rejected when it breaks a "Previous phase" reference; an impact-warning-gated change to an in-progress phase's reviewer count preserving existing decisions; and a Reviewer/Validator/Guest 403 on every Settings endpoint.
- [ ] **Do not** implement the Jobs-behavior-described revision/activation job, protocol-version comparison, or rollback UI — the Description and most of UI/API behavior explicitly exclude them, and Acceptance criteria explicitly says "no protocol-revision draft/version-comparison/activation/rollback/change-impact UI is present." Treat the Jobs behavior field and the "Save in a settings dialog" line in Acceptance criteria as the unresolved contradictions they are (see Gaps) rather than silently implementing either.
- [ ] Report which commands were run and their actual results, and explicitly record which side of each contradiction below was chosen (or that neither was implemented pending a decision).

## Gaps and pending decisions

Explicit conflicts, all present verbatim in the fields captured above: Owner-only versus Manager tab access (Users and permissions says only `PROJECT_OWNER`, UI behavior's Access rules extends Screening/QA/Extraction/Reporting to Project Manager); inline editing (UI behavior) versus "Save in a settings dialog" (Acceptance criteria); no revision workflow (Description/most of API behavior) versus revision/activation jobs (Jobs behavior); Owner-only export (UI behavior) versus a possible broader Admin reading of "audited global access" (Users and permissions) and an imprecise Acceptance-criteria reference to "data available in the Control DB" that must not cause inadvertent protocol duplication. QA validation-workflow scope and Bubble chart availability depend on the global decisions tracked in [pending decisions](PENDING-DECISIONS.md) items 10 and 13. Implement only the direct-editing core described consistently across Description/most of UI/API/Data model, and request a decision before building anything tied to these contradictions.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [21.01 — Reuse wizard contracts/controls and organize General, Screening, QA, Data extraction, and Reporting](sub-issues/21-01.md)
- [21.02 — Apply the detailed UI/API matrix: General for Owner/Admin](sub-issues/21-02.md)
- [21.03 — Implement inline Add/Edit/Save/Cancel/confirmed Delete](sub-issues/21-03.md)
- [21.04 — Manage questions, sources/strategies, and JSON export](sub-issues/21-04.md)
- [21.05 — Manage screening defaults, criteria, and phases](sub-issues/21-05.md)
- [21.06 — Allow rule changes after work starts with an impact warning and confirmation, preserving…](sub-issues/21-06.md)
- [21.07 — Manage QA, categories/subcategories/lists/dependencies, and charts through fixed-schema CRUD](sub-issues/21-07.md)
- [21.08 — Keep createdBy/createdAt/modifiedBy backend-managed](sub-issues/21-08.md)
- [21.09 — Audit mutations and return field errors](sub-issues/21-09.md)
