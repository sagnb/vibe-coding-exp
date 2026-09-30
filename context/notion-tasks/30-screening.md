# 30 — Implement Screening Assignment and Reviewer Decision Flow

## Source and usage

- Source: [Implement Screening Assignment and Reviewer Decision Flow](https://app.notion.com/p/Implement-Screening-Assignment-and-Reviewer-Decision-Flow-3b349936029380d8bb08efcfde18e9a5).
- Notion ID: `3b349936029380d8bb08efcfde18e9a5`.
- Source domain: Screening.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide the Screening workspace for a project's configured screening phases. Project Owners and Project Managers can open and close screening phases, configure the reviewer count for the active phase, and assign eligible papers. All non-Guest project members can access an open phase workspace, see assignments, and complete their own assigned screening work.
- **Functional Domain:** Screening
- **Behavior:** Legacy ReLiS assigns papers from all papers or previous-phase output, supports auto/manual reviewer assignment, shows pending/completed lists, lets reviewers select the next paper, and captures inclusion/exclusion criteria. NeoReLiS keeps the workflow but uses the current phase configuration and shared assignment kernel.
- **Users and permissions:**
  - Project Owner, Project Managers, Application Administrators can: open and close a screening phase; create bulk screening assignments; change the reviewer count for the current phase; view all assignments and all completed screenings; cancel an assignment.
  - Project Owner, Project Managers, Reviewers, and Validators can: access an open screening phase workspace; view all assignments for that phase; view all completed screenings for that phase; view their own assignments and pending work; submit and edit only their own screening decisions while the phase is open.
  - Guests cannot access screening phase workspaces, assignments, or screening decisions.
  - Every non-Guest project member is eligible for normal screening assignment in this ticket. Validator eligibility may later be restricted by a project policy without changing the assignment data model.
- **UI behavior:**
  - Every table in this ticket supports: a search field across relevant available values (paper key, title, assigned user, assigned-by user, decision, status); filters appropriate to the displayed data (assignment status, screening outcome, assigned user, assigned mode, conflict state); sorting by the relevant columns (title, assigned date, screened date, assigned-by user, status/decision); pagination when the result set exceeds the configured page size.
  - **Screening landing page** — lists every configured screening phase in its configured order. For each phase: `Phase Name | My Completion | overall completion | State | Actions`. Phase state is `Open` or `Closed`. Project Owners/Managers/authorised Application Administrators can select Open phase (when closed) or Close phase (when open). When open, show "Open workspace" to all project members. Guests have read-only access to every page in the phase workspace. Closing a phase preserves its assignments/decisions but prevents further screening decisions until reopened. *Note: for this ticket, My Completion and Overall completion are not calculated yet.*
  - **Phase workspace** contains: Phase dashboard; Papers; Screen; Assignments; Screening decisions.
  - **Papers** — tabs All papers | Included | Excluded | Pending | Under review | In conflict; each a paginated table of Papers available in the current screening phase: Paper key | Title | Link | Decision | Source. Title opens the existing read-only Paper details page from Paper Library. The current phase-paper status is derived from submitted screening decisions and the phase's conflict configuration.
  - **Screen** — opens the connected user's next pending assignment for the current phase. Displays only the configured fields for the current phase. Highlights Search Query terms configured for the Paper's Source when applicable. Provides: Include and Exclude actions; inclusion-criteria controls only when Inclusion Criteria are configured; exclusion-criteria controls only when Exclusion Criteria are configured; an optional note field; an "Edit paper" action opening the existing Paper edit page from Paper Library; Save and next.
    - Rules: an Exclusion Criterion is required when excluding a Paper and Exclusion Criteria are configured. Inclusion controls follow the configured inclusion mode: None (no display); One (exactly one required); Any (multiple allowed, at least one required); All (display configured criteria, require a Yes confirmation that all are met). Show "Edit paper" only when the current user is authorised under Paper Library permissions. A user can submit/edit only their own screening assignment. When no pending assignment exists, show a clear empty state, or a 100% progress bar and completion message when all assigned Papers have been screened.
  - **Assignments** — tabs My pending | My assignments | All assignments.
    - My pending: connected user's still-pending assignments — Title | Assigned by | Assigned at | Assigned mode.
    - My assignments: all active assignments of the connected user — Title | Assigned by | Assigned at | Assigned mode.
    - All assignments: all active assignments for the current phase — Cancel | Title | Assigned to | Assigned by | Assigned at | Assigned mode; every project member can view; Assigned mode shows Automatic or Manual; Cancel shown only to Project Owner/Managers/authorised Application Administrators. "Assign papers" shown top-right only to Project Owner/Managers/Application Administrators.
    - Assign papers: opens the bulk assignment page for the current open phase. Displays: a multi-select member input (selected members as chips; opening it shows members with their role in secondary text; Guests excluded); a "Reviewers per paper" input (initialised from current phase config; saving updates the phase's reviewer configuration going forward only, not existing assignments); an "Assign all eligible papers" option (enabled by default; when disabled, a numeric input for the number of papers); a preview table Paper key | Title containing only papers eligible for assignment in the current phase.
  - **Screening decisions** — tabs My screenings | All screenings.
    - My screenings: connected user's completed screenings for the current phase, with a pencil action per row opening the same Screen form prefilled for that paper; editable only while the phase is open and only by that user.
    - All screenings: completed decisions for all users in the current phase — Title | Screened by | Decision | Screened at.
- **API behavior:**
  - **Phase access** — operations to: list screening phases in configured order; retrieve an open phase workspace and its effective configuration; open or close a phase. For this ticket, return `null` for `myCompletion` and `overallCompletion`. Only Project Owner/Managers/Application Administrators can open/close a phase; closing preserves data but blocks all screening changes. All project members, including Guests, may access an open phase workspace (Guests read-only).
  - **Table queries** — paginated queries for: phase Papers; My pending assignments; My assignments; All assignments; My screenings; All screenings — each supporting relevant search/filters/sorting/pagination within the current project and phase.
  - **Phase Papers** — return only Papers available in the current phase; support All/Included/Excluded/Pending/Under review/In conflict; return the derived phase outcome, Paper link, and Paper Source.
  - **Assignments** — return eligible project members and their roles (Guests cannot be assigned); all project members can view assignment data; only Project Owner/Managers/Application Administrators can create or cancel assignments. Operations: preview eligible Papers for assignment; create bulk assignments; cancel an assignment. For bulk assignments: accept selected members, reviewers per Paper, and either all eligible Papers or a requested number; use all active project Papers for the first phase, and Included Papers from the previous phase for later phases; update the current phase reviewer count for future assignments only; preserve existing assignments; prevent duplicate active assignments. Each assignment stores an assignment type — assignments created in this ticket always use `Normal`.
  - **Screening decisions** — operations to: retrieve the caller's next pending assignment; retrieve one assigned Paper using the current phase display configuration; submit or update the caller's screening decision; retrieve the caller's screening progress; list My screenings and All screenings. Return configured search-query terms for the current Paper Source when applicable. When saving a decision: require an active assignment and an open phase; validate inclusion/exclusion criteria; enforce the configured inclusion mode (None/One/Any/All); accept an optional note; update the assignment and recalculate the phase outcome. A user may edit only their own screening decision; Guests cannot submit or edit decisions.
- **Data model:** (stored in the Project DB; user IDs are logical references to Control DB users)
  - `ScreeningPhase` — title, description, order, state (Open/Closed); source-Paper rule; final-phase flag.
  - `ScreeningPhaseConfiguration` — phase reference; reviewers per Paper; displayed Paper fields; inclusion mode and criteria; exclusion criteria; conflict settings.
  - `ScreeningAssignmentRun` — bulk-assignment request, phase, initiating user, date, and reviewers-per-Paper snapshot.
  - `ScreeningAssignment` — phase, Paper, assignee, assigned by, optional assignment-run reference; assignment mode (Automatic/Manual); assignment type (Normal); state (Pending/Completed/Cancelled); timestamps and cancellation info. Only one active assignment may exist for the same Paper, phase, and assignee.
  - `ScreeningDecision` — assignment and Paper reference; decision (Included/Excluded); optional note; submitted/updated timestamps; active or revoked state.
  - `ScreeningDecisionCriterion` — decision reference; criterion reference; criterion type (Inclusion/Exclusion).
  - `ScreeningPaperOutcome` — unique Paper and phase reference; derived state (Pending/Under review/Included/Excluded/In conflict); last recalculation time.
  - Audit events — phase opening/closing; assignment creation/cancellation; decision submission/update/revocation; reviewer-count changes.
- **Acceptance criteria:**
  - [ ] Project Admins can assign eligible papers to the configured number of eligible reviewers for an active phase.
  - [ ] Reviewers can process the next assigned paper and submit the required include/exclude decision and criteria.
  - [ ] A reviewer cannot submit for another user, a closed phase, or an unassigned paper.
  - [ ] Configured previous-phase output and displayed fields determine screening input/context.
  - [ ] Submitted decisions drive paper status, conflict detection, and progress recalculation.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Implement a phase-scoped Screening workspace, assignments, and users' own decisions.

## Proposed prerequisites

- [21 — Implement Direct Project Settings Management](21-project-settings.md)
- [22 — Implement audit and undo UI](22-audit-undo.md)
- [24 — Implement Project Paper List](24-paper-list.md)
- [28 — Provide one CSV Import page organised into progressive sections.](28-csv-import.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Resolve effective configuration and Open/Closed state on the server; list phases and return `null` `myCompletion`/`overallCompletion` at this stage.
2. Create Dashboard/Papers/Screen/Assignments/Screening decisions navigation and tables with search, filters, sorting, and pagination.
3. List Papers as All/Included/Excluded/Pending/Under review/In conflict; reuse Paper Library details.
4. Owner/Manager/Admin previews and assigns to non-Guest members with reviewers per Paper and all/count; snapshot the count, Automatic/Manual mode, Normal type, and Paper/phase/assignee uniqueness.
5. Use active Papers in the first phase and permitted previous-phase output thereafter; changing reviewer count must not change existing assignments.
6. Create Screen next pending, configured fields, Source query highlighting, note, and Save and next. Validate None/One/Any/All inclusion and exclusion criteria when configured.
7. Only the assignee edits their own decision in an open phase; update assignment, outcome, and history. Cancellation/closure preserves data.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the very extensive UI behavior/API behavior/Data model fields.

- [ ] A screening phase's Open/Closed state and effective configuration are resolved server-side; opening/closing is restricted to Project Owner/Managers/Application Administrators, and closing preserves all data while blocking further screening changes.
- [ ] Guests can open a phase workspace **read-only** — they see Papers/Assignments/Screening decisions tables but no Screen action, no Assign papers, and no ability to submit/edit a decision (verify both UI hiding and API rejection).
- [ ] Every listed table (Papers, My pending, My assignments, All assignments, My screenings, All screenings) supports its documented search fields, filters, sorting columns, and pagination.
- [ ] Phase Papers tabs (All/Included/Excluded/Pending/Under review/In conflict) reflect the Paper's **derived** phase outcome from submitted decisions and the phase's conflict configuration — not a manually set status field.
- [ ] Bulk assignment: only Project Owner/Managers/Application Administrators can preview/create/cancel assignments; the eligible-member list excludes Guests; the preview table only shows Papers actually eligible for the current phase (all active Papers for the first phase, Included Papers from the previous phase for later phases); changing "Reviewers per paper" during assignment updates the phase's config for **future** runs only and never mutates already-created assignments.
- [ ] Only one active `ScreeningAssignment` may exist for the same (Paper, phase, assignee) triple — verified with a test that a duplicate bulk-assignment run does not create a second active assignment for an already-assigned pair.
- [ ] The Screen action opens the caller's own next pending assignment (never another user's), shows only the phase's configured displayed fields, highlights the Paper Source's configured search-query terms when applicable, and enforces the configured inclusion mode exactly: None (no inclusion controls shown), One (exactly one required), Any (≥1 of several), All (every configured criterion confirmed via a Yes control) — plus a required Exclusion Criterion whenever excluding and Exclusion Criteria are configured.
- [ ] A user can submit or edit only their **own** screening decision, and only while both the assignment is active and the phase is Open — an attempt against another user's assignment, an unassigned Paper, or a closed phase is rejected server-side.
- [ ] Submitting a decision updates the assignment, recalculates the Paper's phase outcome (including conflict detection per the phase's configured conflict rules), and is reflected immediately in the Papers tabs and Screening decisions lists.
- [ ] "Edit paper" from the Screen form is shown only when the current user is separately authorised under Paper Library permissions — this feature does not grant paper-editing rights by itself.
- [ ] Cancelling an assignment (Owner/Manager/Admin only) preserves audit history and removes it from the assignee's active queue without deleting the assignment's history.
- [ ] `myCompletion`/`overallCompletion` are returned as `null` in this ticket — do not compute placeholder or partial progress numbers; that belongs to [32 — Screening Progress Dashboard and Agreement Statistics](32-screening-metrics.md).
- [ ] Automated tests cover: bulk assignment eligibility (first phase vs. later phase using previous-phase Included output); each inclusion mode's validation; a rejected cross-user decision submission; a rejected decision on a closed phase; and outcome recalculation after a decision is submitted/edited.
- [ ] Report which commands were run and their actual results, and explicitly flag the Guest-access and phase-eligibility contradictions below rather than silently resolving them.

## Gaps and pending decisions

Users and permissions prohibits Guests from the screening workspace, while the UI/API behavior fields on this same card describe Guest read-only access — resolve consistently before releasing access, per [pending decisions](PENDING-DECISIONS.md) item 8. [21 — Project Settings](21-project-settings.md) supports other phase source-status options (All/Included/Excluded), while this ticket's bulk assignment restricts eligible input to "Included Papers from the previous phase" for non-first phases — reconcile effective eligibility per [pending decisions](PENDING-DECISIONS.md) item 9. Paper editing (used by the "Edit paper" action) is a dependency without a complete ticket in the 42-page inventory (see the [notion-tasks README](README.md), "Inventory limitations").

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [30.01 — Resolve effective configuration and Open/Closed state on the server](sub-issues/30-01.md)
- [30.02 — Create Dashboard/Papers/Screen/Assignments/Screening decisions navigation and tables with search, filters,…](sub-issues/30-02.md)
- [30.03 — List Papers as All/Included/Excluded/Pending/Under review/In conflict](sub-issues/30-03.md)
- [30.04 — Owner/Manager/Admin previews and assigns to non-Guest members with reviewers per Paper and all/count](sub-issues/30-04.md)
- [30.05 — Use active Papers in the first phase and permitted previous-phase output thereafter](sub-issues/30-05.md)
- [30.06 — Create Screen next pending, configured fields, Source query highlighting, note, and Save and next](sub-issues/30-06.md)
- [30.07 — Only the assignee edits their own decision in an open phase](sub-issues/30-07.md)
