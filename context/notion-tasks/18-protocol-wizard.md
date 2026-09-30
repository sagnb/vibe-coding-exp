# 18 — Build the Native Project Protocol Wizard

## Source and usage

- Source: [Build the Native Project Protocol Wizard](https://app.notion.com/p/Build-the-Native-Project-Protocol-Wizard-3ad4993602938018a7e5d9d1f1da71c4).
- Notion ID: `3ad4993602938018a7e5d9d1f1da71c4`.
- Source domain: Project & Protocol Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist. This is the largest and most detailed source card in the set — read it in full before implementing any slice.
- Source `Blocked by` relation: Prepare and Provision an Isolated Project Database Template.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Replace DSL/Xtext and external-editor project configuration with a native, autosaved React wizard that collects, validates, and creates the complete review protocol. After confirmation, the same feature provisions the isolated Project DB and persists the initial runtime configuration (`v1`).
- **Functional Domain:** Project & Protocol Management
- **Behavior:** Legacy ReLiS imports PHP/editor-generated configuration that controls screening, QA, extraction, and reporting. ReLiS2.0 replaces this with a native structured form. Its validated payload is stored as normalized configuration metadata in the isolated Project DB and interpreted by reusable application code at runtime.
- **Users and permissions:**
  - Users with project-creation permission can start and submit a project-creation draft.
  - Drafts belong to their creator; Application Administrators may access them for support purposes.
  - After successful provisioning, the creator becomes Project Owner of the new project.
  - Reviewers, Validators, and Guests cannot create or edit project-creation drafts.
  - Changes to an active project protocol are out of scope for this ticket and require a future Project Protocol Revisions workflow.
  - No frontend user can execute raw SQL, generate database schema, or write directly to Control DB or Project DB tables.
- **UI behavior:**
  - On the application home page, provide a Create project action with two choices: Create from scratch; Import configuration JSON.
  - Create from scratch opens an empty Project Creation Wizard.
  - When the user selects Import configuration JSON: display a file-upload dialog for one project-configuration JSON file; show the selected file name and an import-loading state; when the file is valid, open the normal Project Creation Wizard with the imported configuration values prefilled; display a concise import summary and any compatibility warnings at the top of the wizard; allow the user to review and modify every imported field; if the file is invalid or unsupported, show actionable errors and do not open or modify a creation draft; importing a file does not create a project — the user must complete the wizard and explicitly confirm creation.
  - Provide a step-based Project Creation Wizard with autosaved drafts, resume/discard actions, per-step validation, a final review screen, and an explicit Create project confirmation. Whether the wizard starts empty or from an imported JSON configuration, it follows the same validation, autosave, review, confirmation, and provisioning flow.

  **1. Project identity** — Project title (required); Project short name (required and unique; suggest a value from the title); optional description; optional research questions — ordered, repeatable entries, each with an optional description field.

  **2. Modules** — enable/disable switches for: Paper Library & Import; Screening; Screening validation; Quality Assessment; Data Extraction; Reporting / Custom Chart Reports (enabling this also requires Data Extraction).

  **3. Import paper** (when Paper Library & Import is enabled) — an optional, repeatable list of paper sources (each with Value — required, Description, Method — required, Search Query); an optional, repeatable list of search strategies (each with its own description); optional label to define a prefix of the paper key.

  **3(bis). Screening** (when Screening is enabled) — Reviewers per paper (required, int); Conflict type: decision or criteria (displayed and required when reviewers > 1); Conflict resolution: unanimity or majority (required when reviewers > 1); Exclusion criteria — optional, repeatable (criteria + optional description); Screening inclusion mode (All, one, any, none); Use kappa: yes/no; Inclusion criteria — optional, repeatable (criteria + optional description); Screening phases — at least one required, each with Title, Description, and Fields to display (Title, Abstract, Preview, BibTeX, URL — multiple choice).

  **4. Screening validation** (when enabled) — Percentage of excluded papers to validate; Validation outcome handling (Info, Normal, or Veto); Assign papers to validators who have not screened them (yes/no); Screening status to validate (Included, Excluded, Included & excluded).

  **5. Quality Assessment** (when enabled) — Ordered QA questions; response label; Minimum total score threshold; optional QA validation settings: enable validation of completed QA assessments (disabled by default), and when enabled, a validation percentage.

  **6. Data Extraction** (when enabled) — a category builder. Each category has: Category Type (Simple, List, DynamicList, Dependent DynamicList); Required or optional; Category name (required, unique per project); Label name; Maximum number of selectable values; optional Default value.
  - When Simple: Value type (boolean, integer, real, date, string, text); optional validation pattern (RegEx); Maximum text length (optional, default 20); the default-value input adapts to the Value Type.
  - When List: list options (at least two distinct values required).
  - When DynamicList: optional initial values.
  - When Dependent DynamicList: Source DynamicList category (dropdown of existing List categories where possible).
  - Every category can contain subcategories via a "+" action.

  **7. Custom Chart Report** (when enabled) — Required dimension label: "Simple" (one dimension) or "Compare" (two).
  - Simple: chart name (default `chart_name`); category for frequency reporting (dropdown of existing categories); chart_type (Line, Pie, Bar — multi-select).
  - Compare: chart name (required, unique); X-axis category; Y-axis category; chart_type (Line, Pie, Bar, Bubble chart — multi-select).

  **8. Review and confirmation** — a readable summary of all enabled modules and configured fields; blocking validation errors and concise warnings. After final confirmation, show only creation status: Validating, Provisioning project database, Ready, Failed. On failure, show a human-readable error summary and a safe retry action. Do not show protocol-version comparison, activation, rollback, or change-impact screens in this ticket.

- **API behavior:**
  - Use Hono only as the HTTP/API layer. Hono validates commands, calls application services, and returns API responses. Hono must not generate Prisma schemas, migrations, SQL, database tables, columns, views, procedures, or per-category structures from protocol input.
  - Provide authenticated use cases to: validate an imported JSON configuration; create, retrieve, list, autosave, resume, and discard creation drafts; validate a wizard step or a full project-creation command; submit an idempotent project-creation command; retrieve provisioning status; retry an authorized failed provisioning workflow; export an authorized project's active protocol configuration as portable JSON.
  - **JSON configuration import and export** — accept one JSON configuration file through an authenticated endpoint; validate a versioned configuration envelope before processing its content; parse imported JSON as untrusted input only; never interpret imported JSON as SQL, executable code, a Prisma schema, a migration, a database blueprint, a file path, or database connection configuration; normalize a valid imported configuration into the same shared `ProjectCreationDraft` contract used by the wizard; return either validated configuration values suitable for prefilling the wizard, structured field-level errors, unsupported-format-version errors, or non-blocking compatibility warnings; JSON import must not create a Project record, membership, provisioning job, or Project DB; export JSON from the normalized active protocol configuration, never internal database IDs or runtime data.
  - **Shared validation contract** — one shared server-side Zod contract for project-creation drafts, imported configurations, step validation, and final submissions; validate only relevant completed fields for a partial draft or step validation, and the entire protocol on final confirmation; return structured field-level errors and non-blocking compatibility warnings in a client-friendly format; final server-side validation remains authoritative, including project-short-name uniqueness (enforced with a Control DB constraint, not only client-side).
  - **Drafts** — belong only to their authenticated creator unless an Application Administrator has audited global access; drafts are not projects and must never trigger Project DB provisioning; store each draft's current payload as a JSONB document in the Control DB; use the same versioned JSON contract for draft autosave, draft export, and JSON import.
  - **Protocol validation rules** — Screening Validation requires Screening; Custom Chart Reports require Data Extraction and valid referenced extraction categories; Source and Search Strategy tracking require Paper Library & Import; when that tracking is disabled, the corresponding lists must be empty; conflict configuration is required only when reviewers per paper > 1; Paper Source and Search Strategy lists are optional, unique, ordered; the system does not execute searches or connect to declared sources/strategies. Data Extraction/Classification rules use fixed domain models: category names unique within the project; a dependent Dynamic List references an existing Dynamic List category; the backend derives inherited reference names (the client cannot override them); list categories require at least two distinct options; report definitions reference existing eligible extraction configuration keys; category dependencies and reports use portable configuration keys in draft/JSON input, resolved to Project DB records only during protocol persistence; no category/question/phase/source/strategy/report/subcategory ever creates a table, column, Prisma model, migration, SQL statement, view, or procedure.
  - **Paper-key generation** — treat the optional paper-key prefix as an immutable `v1` display setting; allow an empty prefix; do not accept a user-managed paper serial/counter; generate any fallback paper reference atomically in the Project DB via a fixed PostgreSQL sequence or counter table already present in `relis_project_template`.
  - **Idempotent project creation** — on final confirmation, accept an idempotent command using an idempotency key; repeated requests with the same key return the existing project/provisioning result; repeated requests must not create duplicate Control DB projects, `PROJECT_OWNER` memberships, provisioning jobs, or Project databases. In one Control DB transaction: create the Project record with state `PROVISIONING`; set `createdByUserId` to the authenticated creator; reserve the unique project short name; create the creator's protected `PROJECT_OWNER` membership with state `PENDING_PROVISIONING`; create one provisioning job/outbox record linked to the Project and idempotency key. `PROJECT_OWNER` is protected backend state: assigned automatically to the creator; cannot be assigned/removed/changed by membership-management endpoints; the backend enforces it corresponds to `Project.createdByUserId`.
  - **Provisioning behavior** — this ticket depends on [16 — Prepare and Provision an Isolated Project Database Template](16-project-template.md); Hono delegates physical database work to a provisioning service/worker; the Control DB transaction and Project DB work form an idempotent provisioning workflow, not one distributed transaction; generate the Project DB identifier from the internal Project UUID, never the title/short name; maintain `relis_project_template` outside the end-user flow; clone it to create the isolated Project DB; verify the clone is reachable and compatible with the running Prisma data model; never run `prisma migrate deploy`, `prisma db push`, generated SQL, or any schema change during creation; never create tables/columns/views/procedures/models/migrations from protocol categories/questions/phases/sources/strategies/classification configuration/report definitions; use Prisma only for normal CRUD against the cloned DB's existing fixed schema; persist initial protocol version `v1` as normalized records (project settings/enabled modules; Paper Sources and Search Strategies; screening configuration/phases/criteria/reviewer/conflict/validation settings; QA questions/options/scoring/threshold; Data Extraction categories/options/dependencies/multi-value definitions/subcategories; Classification categories and subcategories; reporting and Custom Chart Report definitions); an optional canonical JSONB snapshot of `v1` may be retained for audit only, never to drive database creation/structure/authorization.
  - **Provisioning retry and status** — make each stage idempotent; a retry after failure reuses an already-cloned Project DB, and does not re-insert configuration rows if `v1` was already persisted; activate the Project and the creator's protected `PROJECT_OWNER` membership only after the Project DB is available and `v1` has been persisted successfully; expose only `validating`/`provisioning`/`ready`/`failed` to the frontend; on failure, return a safe human-readable summary and a retryable/non-retryable status; never return database credentials, connection strings, SQL errors, stack traces, database names, or internal infrastructure details; a retry resumes the same workflow and never creates another Project.
- **Data model:**
  - Control DB: project-creation drafts, draft owner, current wizard step, autosaved draft payload, project registry, lifecycle/provisioning state, Project DB reference, creator membership, and provisioning-job reference.
  - Project DB: immutable initial protocol snapshot (`v1`), module toggles, project settings, screening phases, exclusion criteria, separate project-scoped `PaperSource` and `SearchStrategy` reference records seeded from the initial protocol configuration (imported papers retain nullable references to the selected source/search strategy), QA questions and shared response scale, extraction category definitions, choice-list definitions, category dependencies/subcategories, custom report definitions, and a creation audit record.
  - Project DB schema uses static Prisma migrations only; configured extraction categories are metadata interpreted at runtime, never dynamically generated columns or tables.
- **Acceptance criteria:**
  - [ ] A user can create, resume, validate, review, and submit a complete project protocol without DSL/Xtext or ReLiS Editor access.
  - [ ] The wizard covers all legacy project-configuration categories listed in this ticket.
  - [ ] Conditional fields and cross-step validation prevent invalid protocol combinations.
  - [ ] Draft state survives refresh and can be resumed only by its owner or an authorized Application Administrator.
  - [ ] A confirmed valid draft creates exactly one isolated Project DB and one initial protocol configuration version (`v1`).
  - [ ] The initial configuration controls modules, screening settings, QA template, extraction form, and custom report definitions through normalized metadata.
  - [ ] The project creator becomes Project owner only after provisioning succeeds.
  - [ ] No DSL import, generated SQL, generated table, generated view, or stored procedure is used.
  - [ ] Failed provisioning leaves no active project and returns actionable error information.
  - [ ] Repeating the same confirmation request does not create duplicate projects or databases.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Prepare and Provision an Isolated Project Database Template
- **Jobs behavior:** Run Project DB provisioning and initial configuration persistence through an idempotent workflow. Track `validating`, `provisioning`, `ready`, and `failed` states. A failed attempt must never leave an active project pointing to a missing or incomplete Project DB. Retrying the same confirmed draft must safely recover or reuse the same provisioning attempt.
- **Audit:** Empty

## Objective and summarized requirements

Replace the DSL/editor project configuration flow with a native autosaved wizard that validates, creates, and persists the complete v1 protocol, then provisions the isolated Project DB.

## Proposed prerequisites

- [16 — Prepare and Provision an Isolated Project Database Template](16-project-template.md)
- [15 — Implement Project Context, Roles, and Membership](15-membership.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Implement the shared Zod `ProjectCreationDraft` contract covering every wizard step (identity, modules, import/sources, screening, validation, QA, extraction, custom charts) and reuse it for autosave, JSON import, step validation, and final submission.
2. Build the JSON-import flow as prefill-only: validate the envelope, normalize into the shared contract, and never create a Project/membership/job from an import alone.
3. Implement autosaved, resumable drafts scoped to their creator (plus audited Application Administrator access), with per-step and full-protocol validation and the protocol validation rules (Screening Validation requires Screening; Custom Chart Reports require Data Extraction; source/strategy lists require Paper Library & Import; conflict fields required only when reviewers > 1; etc.).
4. Implement the idempotent final-confirmation command: one Control DB transaction creating the Project (`PROVISIONING`), reserving the short name, creating the protected `PENDING_PROVISIONING` `PROJECT_OWNER` membership, and one provisioning job/outbox record.
5. Delegate physical provisioning to [16](16-project-template.md); never run migrations/db push/generated SQL from this feature; persist `v1` as normalized rows only after the cloned database is verified.
6. Expose only `validating`/`provisioning`/`ready`/`failed` to the frontend, with safe error summaries and a retry action; activate the project and membership only after both the database and `v1` are confirmed persisted.
7. Make every stage idempotent (draft autosave, submission, provisioning, retry) so repeated requests never duplicate projects, memberships, jobs, or databases.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the very extensive UI behavior/API behavior/Data model fields.

- [ ] A user with project-creation permission can complete the wizard (identity → modules → conditional sections → review) and confirm creation, with zero DSL/Xtext/ReLiS-Editor involvement.
- [ ] Every module-conditional section (Import paper, Screening, Screening validation, QA, Data Extraction, Custom Chart Report) appears only when its owning module toggle is enabled, and the cross-module rules are enforced server-side: Screening Validation requires Screening; Custom Chart Reports require Data Extraction (and valid referenced categories); Source/Search Strategy tracking requires Paper Library & Import.
- [ ] Draft autosave persists the current step's payload as JSONB in the Control DB, survives a page refresh, and can be resumed only by its creator or an audited Application Administrator — never by an arbitrary authenticated user.
- [ ] JSON configuration import validates the envelope, normalizes into the same `ProjectCreationDraft` contract, prefills the wizard, and creates **no** Project/membership/job/database by itself — only explicit wizard confirmation does that.
- [ ] Field-level validation runs per-step (partial) and again in full on final confirmation; the project short name's uniqueness is enforced by a Control DB constraint, not only client-side.
- [ ] Final confirmation is idempotent via an idempotency key: repeating the same confirmed request returns the existing project/provisioning result rather than creating a duplicate Project, `PROJECT_OWNER` membership, provisioning job, or Project database.
- [ ] The creator's `PROJECT_OWNER` membership is created as `PENDING_PROVISIONING` in the same Control DB transaction as the Project record, and is protected: it cannot be assigned, removed, or changed through the normal membership-management endpoints from [15](15-membership.md), and the backend enforces `PROJECT_OWNER` corresponds to `Project.createdByUserId`.
- [ ] No step of this feature runs `prisma migrate deploy`, `prisma db push`, generated SQL, or creates any table/column/view/procedure/model/migration from protocol input — every category, question, phase, source, strategy, and report definition is persisted as configuration **rows** in the already-fixed schema from [04](04-project-prisma.md)/[16](16-project-template.md).
- [ ] The optional paper-key prefix is stored as an immutable `v1` display setting (empty allowed); the backend generates any fallback paper reference atomically via a fixed sequence/counter already present in the template — never a user-managed counter.
- [ ] The frontend only ever observes `validating` / `provisioning` / `ready` / `failed`; on failure it gets a safe, human-readable summary and a retry action — never a database credential, connection string, SQL error, or stack trace.
- [ ] The project and its owner membership are activated (moved out of `PROVISIONING`/`PENDING_PROVISIONING`) only after both the Project DB is confirmed ready and protocol `v1` has been fully persisted — never before.
- [ ] Retrying a failed or interrupted provisioning reuses the already-cloned database (per [16](16-project-template.md)) and does not re-insert `v1` configuration rows if they were already persisted — verified with a test that kills the flow mid-way and retries it.
- [ ] Automated tests cover at minimum: a full happy-path submission producing exactly one project/database/`v1` snapshot; a repeated (duplicate) confirmation with the same idempotency key producing no duplicates; a validation failure blocking submission; a JSON-import prefill that creates no project; and a simulated provisioning failure surfacing a safe retryable error.
- [ ] Report which commands were run and their actual results, explicitly noting any acceptance criterion or validation rule left unverified because [16](16-project-template.md) or [15](15-membership.md) were not yet available, given the circular dependency recorded in [pending decisions](PENDING-DECISIONS.md).

## Gaps and pending decisions

There is an explicit circular dependency with [16 — Prepare and Provision an Isolated Project Database Template](16-project-template.md) (each blocks the other). The guide separates the shared contract, template/cloning, and orchestration; template preparation/migration belongs to the release process, and project creation only clones a ready schema and writes `v1` — do not run migrations in the end-user flow. See [pending decisions](PENDING-DECISIONS.md) items 1, 5, and 6 for the related role-model and creation-permission conflicts (who may create projects; `PROJECT_ADMIN` vs. protected `PROJECT_OWNER`).

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [18.01 — Define a versioned Zod contract shared by Control DB JSONB drafts, JSON import, step validation, and final…](sub-issues/18-01.md)
- [18.02 — Implement an empty start or validated JSON that only prefills the wizard](sub-issues/18-02.md)
- [18.03 — Build identity fields: title, unique short name, description, and ordered questions](sub-issues/18-03.md)
- [18.04 — Build Screening: reviewer count, decision/criteria conflicts, unanimity/majority, criteria,…](sub-issues/18-04.md)
- [18.05 — Build QA with ordered questions, shared response scale, scores, and threshold](sub-issues/18-05.md)
- [18.06 — Build Simple (boolean/integer/real/date/string/text), List (at least two options), DynamicList, and…](sub-issues/18-06.md)
- [18.07 — Build Simple/Compare reports referencing valid categories and chart types](sub-issues/18-07.md)
- [18.08 — Implement autosave/resume/discard, final review, field errors, and confirmation](sub-issues/18-08.md)
- [18.09 — In a Control DB transaction, reserve a PROVISIONING project, createdByUserId, a protected pending…](sub-issues/18-09.md)
- [18.10 — Orchestrate template cloning, normalized v1 protocol CRUD, and activation only after success](sub-issues/18-10.md)
- [18.11 — Export portable configuration without internal IDs/runtime data/secrets](sub-issues/18-11.md)
