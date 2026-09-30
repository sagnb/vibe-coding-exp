# 35 — Implement Classification Assignment and Classification Workspace

## Source and usage

- Source: [Implement Classification Assignment and Classification Workspace](https://app.notion.com/p/Implement-Classification-Assignment-and-Classification-Workspace-3b3499360293808c9848e3c2ef6b084b).
- Notion ID: `3b3499360293808c9848e3c2ef6b084b`.
- Source domain: Data Extraction.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide a project-scoped Classification workspace for assigning eligible Papers and recording their classification data. Classification uses a project-specific classification schema built with the same category types and rendering rules as Data Extraction.
- **Functional Domain:** Data Extraction
- **Behavior:** ReLiS enables Classification per project. Included Papers become eligible for Classification; excluded Papers do not. A Project Admin assigns eligible Papers to non-Guest reviewers. Assignees complete a project-specific Classification form with lists, dynamic lists, dependencies, and subcategories. Dynamic-list values are managed per project. Legacy subcategories can only be filled after the first save. A classification can be edited, removed, or the Paper excluded. The new relis keeps the project-specific Classification workflow, eligible-Paper rules, assignments, dynamic reference values, and the ability to edit, remove, reassign, or exclude a Paper. It replaces the legacy generated form and database structure with a native React form backed by the fixed Project DB schema. Subcategory values can be entered directly when creating a Classification; no first save is required.
- **Users and permissions:**
  - Project Owner, Project Managers, and Application Administrators can: assign Papers for classification; view all classification assignments; manage Classification Reference Lists; reassign a current classification assignment; edit or remove a classification when authorised.
  - Project Owner, Project Managers, Reviewers, and Validators can: receive classification assignments; classify their own assigned Papers; view their own assignments and classifications; reassign a paper that they already classified.
  - Guests cannot receive assignments, classify Papers, manage reference lists, or reassign work.
- **UI behavior:**
  - **Classification workspace navigation** — Classification dashboard (actions: "Assign papers" visible only to Owner/Manager/Admin, opening the assignment page; "Classify" visible to eligible users, opening the next pending assignment); Papers; Reference lists; Classify; Assignments; Progress. Do not display Classification when the module is disabled for the project.
  - **Papers** — tabs All papers | Classified | Excluded | Pending; each a searchable/filterable/sortable/paginated table: Paper key | Title | Status. Title opens the Paper in the current Classification context. Status: Classified (active classification); Excluded (excluded from the review); Pending (eligible, no active classification). When Screening is enabled, only Papers finally Included in the last Screening phase are eligible; when disabled, active non-excluded Papers are eligible.
  - **Reference categories** — one tab per Classification DynamicList reference, named by its configured Reference name (or the DynamicList category label when unconfigured); a shared reference name appears only once. Each tab: editable table Value | Description | Actions. Value required+unique within its list; Description optional. Every eligible classification user can add a new row, edit a value inline, or delete an unused value with confirmation. A value already used by a Classification cannot be deleted. These values populate their matching DynamicList controls in the Classification form.
  - **Assignments** — tabs My assignments | My pending assignments | My classified | All assignments; each a searchable/filterable/sortable/paginated table: Paper key | Title | Status | Assigned to | Assigned by | Assigned at. My assignments: all active assignments of the connected user. My pending assignments: active assignments whose Paper has no active Classification. My classified: assignments whose Paper has an active Classification. All assignments: every active classification assignment in the current project. A title opens the Paper in the Classification context. "Assign papers" shown only to Owner/Manager/authorised Admin.
  - **Assign papers** — a multi-select of eligible project members (chips; each option shows name+role; Guests excluded); an "Assign all eligible Papers" option (enabled by default; when disabled, a numeric input for the count); a preview table Paper key | Title of only Papers that are not excluded, are eligible for Classification, have no active Classification, and have no active classification assignment. On confirmation, assigns selected Papers evenly across selected members; each Paper receives one classification assignment.
  - **Classify** — opens the connected user's next pending assignment. Displays: Paper key/title; Abstract; Paper preview; Classification section. When a Classification already exists, shows its saved values in a readable structured layout with authorised Edit/Delete actions. When none exists, shows "Add classification" opening a dialog with the fields defined by the active Classification schema.
  - **Classification form rendering** — Simple (Boolean → Yes/No; Integer/Real → numeric; Date → date picker; String → single-line; Text → multi-line, applying required/default/validation-pattern/max-length); List (single-select when max=1, multi-select otherwise, enforcing configured max); DynamicList (values from the project's Classification Reference List, single/multi per configured max); Dependent DynamicList (control unavailable until its source value is selected; shows only values valid for that selected source); categories with subcategories (collapsible area in the Add Classification dialog rendering every configured subcategory with the same rules — parent and subcategory values may both be provided before the first save). Empty state when no pending assignment.
  - **Reassign classification** — shown on a current Classification Paper to the current assignee, Project Owner, Project Managers, Application Administrators. Form: Assigned to (required; eligible project member, excluding Guests and the current user); Note (optional). On confirmation: closes the current active assignment as Reassigned; creates a pending manual assignment for the selected member; preserves assignment history; keeps any existing Classification visible to the new assignee, who may edit or remove it.
- **API behavior:**
  - Use Hono as the authenticated HTTP/API layer. Enforce the current project context, Classification-module state, and caller permissions for every operation. Resolve the active Classification schema and category rules server-side. Do not generate database tables/columns/Prisma models/SQL/migrations from Classification categories.
  - Operations: retrieve the Classification dashboard and caller's available actions; list Classification Papers by all/classified/excluded/pending; list Classification Reference Categories and their values; add/edit/delete an unused Reference Category value; list My assignments / My pending assignments / My classified / All assignments; preview eligible Papers for a bulk classification assignment; create a bulk classification assignment; retrieve the caller's next pending classification assignment; retrieve a Paper in its Classification context; create/update/remove a Classification; reassign an active Classification assignment.
  - **Classification Papers** — return only Papers of the current project, with search/filters/sorting/pagination; derive status server-side (Classified/Excluded/Pending). A Paper is eligible only when active and not excluded, and: when QA is enabled, it has a final QA outcome of Included; otherwise, when Screening is enabled, it is finally Included in the last Screening phase; otherwise, it is an active non-excluded Paper.
  - **Reference Categories** — only classification-eligible non-Guest members may manage values; require a unique non-empty value within its Reference Category; reject deletion when the value is used by an active Classification; return values only for the requested project Reference Category.
  - **Assignments** — only Owner/Manager/Admin can create bulk assignments; accept selected eligible member IDs and either `assignAll` or a Paper count; exclude Guests from assignment recipients; select only Papers that are eligible, non-excluded, unclassified, and without an active classification assignment; create one active classification assignment per Paper; distribute selected Papers evenly across selected members; use an idempotency key so repeating one confirmation does not duplicate assignments.
  - **Classification form** — return the active Classification schema with user-facing labels, category types, required state, defaults, validation rules, list options, reference values, dependencies, and subcategories; validate every submitted value against this schema; support Simple/List/DynamicList/Dependent DynamicList values plus subcategory values submitted with the initial Classification; a Dependent DynamicList value must be valid for its selected source value; persist through the existing fixed generic Project DB schema.
  - **Classification permissions** — a user may create or edit a Classification only for their own active assignment; Owner/Manager/Admin may manage Classifications where authorised; Guests cannot create, edit, delete, assign, or reassign Classification work.
  - **Reassignment** — allowed only to the current assignee, Owner, Manager, Admin; requires an eligible non-Guest recipient different from the caller; marks the current assignment Reassigned, preserves its history, and creates one new pending manual assignment; may occur even when the Paper already has a Classification; the new assignee receives the existing Classification and may edit/remove it. Record assignment, reassignment, Classification creation/update/deletion in the project audit trail.
- **Data model:** (Project DB)
  - `ClassificationAssignment` — Paper; assignee; assigning actor; assigned time; mode (Automatic/Manual/Reassigned); state (Pending/Completed/Reassigned/Cancelled); optional note.
  - `Classification` — Paper; active state; classified by; created/updated timestamps.
  - `ClassificationValue` — Classification; configured category or subcategory; stored value or selected reference value.
  - `ClassificationReferenceList` and `ClassificationReferenceValue` — reference name; value; optional description; active state.
- **Acceptance criteria:**
  - [ ] Eligible Papers can be assigned evenly to selected non-Guest project members.
  - [ ] A Paper with an active Classification cannot receive another normal classification assignment.
  - [ ] Each assignee can open and classify their next pending Paper.
  - [ ] Simple, List, DynamicList, Dependent DynamicList, and subcategory fields render according to their configured rules.
  - [ ] Subcategory values can be entered while creating the Classification.
  - [ ] DynamicList reference values are managed through the Reference lists page and appear in their matching form controls.
  - [ ] Reassignment preserves history and gives the new assignee access to the existing Classification.
  - [ ] Papers, assignments, and progress are scoped to the current project.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Assign eligible Papers and record classification using generic categories, dynamic references, and reassignment.

## Proposed prerequisites

- [21 — Implement Direct Project Settings Management](21-project-settings.md)
- [24 — Implement Project Paper List](24-paper-list.md)
- [30 — Implement Screening Assignment and Reviewer Decision Flow](30-screening.md)
- [33 — Implement Quality Assessment Assignment and Assessment Flow](33-qa-assessment.md)
- [34 — Implement QA Results, Low-quality Exclusion, and Progress](34-qa-results.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Resolve module/configuration/permissions and eligibility: active, non-excluded, and Included after QA when enabled; otherwise Included in final Screening; otherwise available in the library.
2. Create All/Classified/Excluded/Pending Papers and My/My pending/My classified/All assignments with search, filters, sorting, and pagination.
3. Owner/Manager/Admin previews and distributes evenly among non-Guests; an eligible Paper without active classification/assignment receives one assignment, idempotently.
4. Create Reference lists by configured name or label; avoid duplicate tabs, require unique nonempty values, and block deletion of used values. Eligible non-Guest members manage values according to the API.
5. Create next Paper, details, and authorized Add/Edit/Delete Classification; assignees modify only their own work unless explicit management exceptions apply.
6. Render Simple, List, DynamicList, dependent fields, and subcategories on the first save; validate required/default/regex/max length/selections/dependencies on the server.
7. Persist Classification/ClassificationValue/Assignment/ReferenceList/Value in the fixed schema without generating category-specific structures.
8. Allow reassignment by assignee/Owner/Manager/Admin to another eligible member: close the current assignment as Reassigned, create a pending manual assignment, and preserve classification/history for the new assignee.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the extensive UI behavior/API behavior/Data model fields.

- [ ] Paper eligibility for Classification follows the exact precedence in API behavior: active + not excluded, **and** (QA Included when QA enabled) **else** (finally Included in last Screening phase when Screening enabled) **else** (any active non-excluded Paper) — reconcile against the simplified UI wording noted in Gaps, and implement the API's precedence.
- [ ] Bulk assignment: eligible Papers are those not excluded, eligible, with no active Classification, and no active assignment; Guests are excluded from recipients; assignment is even across selected members; a repeated confirmation with the same idempotency key does not create duplicate assignments.
- [ ] A Paper with an active Classification cannot receive another **normal** classification assignment (Reassignment is the only path to change assignee for an already-classified Paper).
- [ ] The Classify flow opens the caller's own next pending assignment; when a Classification already exists it is shown read/edit per authorization, otherwise "Add classification" opens the schema-driven form.
- [ ] Every category type renders per its configured rules: Simple (correct control per value type, required/default/pattern/max-length applied); List (single vs. multi-select per configured max, enforcing it); DynamicList (values sourced from the project's Reference List, single/multi per max); Dependent DynamicList (disabled until its source value is chosen, then filtered to values valid for that source).
- [ ] Subcategory values can be entered **during** initial Classification creation — no "must save parent first" legacy restriction is reintroduced.
- [ ] Reference Category values are managed only via the Reference lists page by eligible non-Guest classification users; a value already used by an active Classification cannot be deleted; each shared reference name appears as exactly one tab even if used by multiple categories.
- [ ] A user may create/edit a Classification only for their own active assignment; Owner/Manager/Admin may manage Classifications where separately authorised; Guests can never assign, classify, manage reference lists, or reassign.
- [ ] Reassignment: allowed only for the current assignee, Owner, Manager, or Admin, to an eligible non-Guest recipient different from the caller; it closes the current assignment as `Reassigned` (preserving history), creates one new `Pending` manual assignment, and the new assignee can see and edit/remove the Paper's existing Classification.
- [ ] Every relevant action (assignment creation, reassignment, Classification create/update/delete) is written to the project audit trail.
- [ ] All list/assignment/progress data is scoped strictly to the current project — no cross-project leakage.
- [ ] Automated tests cover: eligibility precedence with QA/Screening variously enabled/disabled; a rejected second normal assignment for an already-classified Paper; each category-type's rendering/validation rule; a Dependent DynamicList rejecting a value invalid for its source; a full reassignment preserving history and giving the new assignee access; and a Guest's 403 on every mutation endpoint.
- [ ] Report which commands were run and their actual results, and record how the UI-vs-API eligibility wording conflict (see Gaps) was resolved.

## Gaps and pending decisions

UI behavior's eligibility description is simplified relative to the API behavior's full precedence rule (QA → Screening → library) — implement the more complete API rule and reconcile the UI copy, per [pending decisions](PENDING-DECISIONS.md) item 12. The pending-lists in [36 — the data extraction dashboard and progress](36-classification-progress.md) are based on the mere absence of a Classification record, while that same ticket's own API rule requires the **new** assignee to save/confirm even when a Classification already exists after a reassignment — define assignment state vs. completion clearly so a pre-existing Classification is not confused with the new assignee's confirmed work; do not conflate "record exists" with "new assignee has confirmed."

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [35.01 — Resolve module/configuration/permissions and eligibility: active, non-excluded, and Included after QA when…](sub-issues/35-01.md)
- [35.02 — Create All/Classified/Excluded/Pending Papers and My/My pending/My classified/All assignments with search,…](sub-issues/35-02.md)
- [35.03 — Owner/Manager/Admin previews and distributes evenly among non-Guests](sub-issues/35-03.md)
- [35.04 — Create Reference lists by configured name or label](sub-issues/35-04.md)
- [35.05 — Create next Paper, details, and authorized Add/Edit/Delete Classification](sub-issues/35-05.md)
- [35.06 — Render Simple, List, DynamicList, dependent fields, and subcategories on the first save](sub-issues/35-06.md)
- [35.07 — Persist Classification/ClassificationValue/Assignment/ReferenceList/Value in the fixed schema without…](sub-issues/35-07.md)
- [35.08 — Allow reassignment by assignee/Owner/Manager/Admin to another eligible member: close the current…](sub-issues/35-08.md)
