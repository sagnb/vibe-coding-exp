# 31 — Implement Screening Conflict Resolution and Validation Workspaces

## Source and usage

- Source: [Implement Screening Conflict Resolution and Validation Workspaces](https://app.notion.com/p/Implement-Screening-Conflict-Resolution-and-Validation-Workspaces-3b34993602938049bbc5ff5867cd9e82).
- Notion ID: `3b34993602938049bbc5ff5867cd9e82`.
- Source domain: Screening.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Implement conflict handling and optional screening validation for an open screening phase. Every command must resolve the effective phase configuration server-side: use the phase custom configuration when present; otherwise use the project default screening configuration.
- **Functional Domain:** Screening
- **Behavior:** Legacy ReLiS detects decision and criteria conflicts, offers personal conflict lists, resolves through unanimity/majority/manual decisions, and supports validation modes with validation task lists and results. NeoReLiS shows the same outcomes through one clear, auditable workflow.
- **Users and permissions:**
  - Project Owner, Project Managers, and Application Administrators can: view all conflicts and validation work; create validation assignment runs; resolve unresolved conflicts; view validation results.
  - Validators, Project Owner, and Project Managers can complete validation assignments assigned to them.
  - Reviewers can view conflicts and validation data but cannot resolve or validate unless they also hold an eligible validation role.
  - Guests have read-only access to every conflict and validation page. Guests cannot be assigned, validate, resolve a conflict, or modify any data.
- **UI behavior:**
  - Users access conflicting Papers through the `In conflict` filter in the current phase Papers page. Selecting a Paper title opens the existing Paper details page in the current screening-phase context. In addition to the reference metadata from Paper Library, display a Screening result section.
  - **Screening result** `[Pending | In review | Included | Excluded | In conflict]` — displays the derived phase outcome and an assignment table: Assigned to | Phase | Assignment type | Decision | Criteria | Screened at. When the outcome is In conflict, show a conditional Actions column: Edit for each completed conflicting decision the current user may modify (Owner/Manager/Admin edit any; Reviewer/Validator edit only their own); Guests see no action. Edit opens the existing Screen page for that Paper/assignment, prefilled, and saving recalculates the Paper outcome immediately — if resolved, it leaves the `In conflict` filter. Display "Add reviewer" only to Owner/Manager/Admin while the phase is open: a manual-assignment form/inline row with Assigned to (required, eligible non-Guest member), Assignment type (required: Normal/Veto/Info), and an optional Note; creating it records Manual mode, adds the row to the Screening result table, and recalculates the outcome after its decision is submitted.
  - **Screening history** — read-only section below the result table: User | Decision | Criteria | Paper status | Operation | Time. An entry is appended whenever a decision is submitted/edited/revoked/validated/changed to resolve a conflict (using "Conflict resolution edit" as the operation when applicable). Cancelling an assignment/decision preserves historical entries. All project members, including Guests, can view this history read-only.
  - **Conflict rules** (apply the effective phase configuration): when reviewers per paper is 1, no normal-review conflict can occur; conflict type Decision → different Include/Exclude decisions conflict; conflict type Criteria → different decisions conflict, and all-exclude decisions with different exclusion criteria also conflict; resolved by Unanimity → remains In conflict until every active reviewer agrees on decision (and criterion, when applicable); resolved by Majority → a strict Include/Exclude majority determines the outcome; with conflict type Criteria + resolved by Majority, if the Exclude majority selected different exclusion criteria, the Paper remains In conflict; a tied majority remains In conflict.
  - **Validation workspace** — when Screening Validation is enabled, provide a Validation navigation group in the open phase workspace: Validate; All assignments; Validated paper. All members can view read-only; Reviewers and Guests cannot receive validation assignments or submit validation decisions.
  - **Phase dashboard actions** — when Validation enabled, show a Validation action area: "Assign validation" only to Owner/Manager/Admin; "Validate" only to Owner/Manager/Validators. Selecting "Assign validation" opens the dedicated assignment page; "Validate" opens the caller's next pending validation assignment.
  - **Assign validation** — displays: number of eligible Papers; number already assigned; number that will be assigned. A "Select validator(s)" multi-select (chips; dropdown shows eligible members + role; only Owner/Manager/Validators selectable, Reviewers/Guests excluded; optionally excludes a user who already screened the relevant Paper). A validation-scope choice: "All final excluded Papers" (default) or "Excluded by criteria" (available only when exclusion criteria are configured — with a multi-select of exclusion criteria; only final Excluded Papers associated with at least one selected criterion are eligible). A required "Percentage of eligible Papers" input, initialised from the effective phase validation configuration. A preview table Paper key | Title of only eligible Papers under the selected scope/percentage/validator selection/independence rule. Actions: "Assign validation" (creates assignments) and "Cancel."
  - **Validate** — shown only to Owner/Manager/Validators; opens the caller's next pending validation assignment; reuses the Screen form with only the configured fields; provides Include/Exclude, configured inclusion/exclusion criteria controls, an optional note, and Save and next. The submitted decision is recorded as a validation decision applying the configured validation outcome handling (Info/Normal/Veto). Empty state when no pending assignment.
  - **Validated Paper** — table Paper | Assigned to | Decision | Inclusion Criteria | Exclusion criteria | Screening time.
  - **All Assignments** — Cancel | Title | Assigned to | Assigned by | Assigned at | State | Validation decision; every member can view read-only; Cancel shown only to Owner/Manager/Admin; cancelling removes the assignment from the validator's active queue while preserving audit history, and if a submitted decision exists, requires confirmation, revokes the decision, and recalculates the affected outcome.
- **API behavior:**
  - Resolve the effective phase configuration server-side. Reject every conflict or validation mutation while the phase is closed. Guests have read-only access only.
  - Operations: retrieve a Paper's screening result/assignments/decisions/history in the current phase; edit an eligible conflicting screening decision; create a manual reviewer assignment for a conflicting Paper; list validation assignments and results; preview a validation assignment run; create a validation assignment run; retrieve the caller's next validation assignment; submit/update the caller's validation decision; cancel an eligible validation assignment.
  - For a conflicting decision: Owner/Manager/Admin may edit any eligible conflicting decision; Reviewers/Validators may edit only their own; Guests cannot edit; manual reviewer assignments available only to Owner/Manager/authorised Admin.
  - For validation assignment: accept selected validator IDs; accept scope `all_excluded` or `excluded_by_criteria`; when `excluded_by_criteria`, accept one or more configured exclusion-criterion IDs; accept the selected percentage; initialise the frontend value from the effective phase configuration but do not change that configuration from this command; select only final Excluded Papers from the current phase; when criteria are selected, retain only Papers excluded using at least one selected criterion; exclude Papers already assigned for validation; sample the requested percentage uniformly at random; assign one selected eligible validator per sampled Paper, distributed across selected validators; allow only Owner/Manager/Validators to receive validation assignments (exclude Reviewers/Guests); when enabled, exclude users who screened that Paper in the current phase; reject the command when the selected validators cannot satisfy the effective independence rule. The preview and final assignment must use the same validated selection rules — if eligibility changes before confirmation, return an updated preview rather than silently assigning a different set.
  - For validation decisions: require an active validation assignment belonging to the caller; apply the server-resolved validation type (Info/Normal/Veto); preserve every screening and validation history event.
  - Cancelling a validation assignment: available only to Owner/Manager/Admin; preserves audit history; revokes an existing validation decision only after confirmation.
  - After a screening-decision edit, validation decision, validation cancellation, or manual reviewer assignment, recalculate the Paper's phase outcome and derived conflict state.
- **Data model:** (stored in the Project DB)
  - `ScreeningPaperOutcome` — existing derived Paper status per phase: Pending, In review, Included, Excluded, or In conflict. No separate conflict table.
  - `ScreeningValidationRun` — phase, actor, scope, selected criteria, percentage, configuration snapshot, counts, idempotency key, timestamp.
  - `ScreeningValidationAssignment` — phase, Paper, validator, run, state (Pending/Completed/Cancelled), timestamps.
  - `ScreeningValidationDecision` — assignment, Include/Exclude decision, criteria, note, submitted/updated/revoked timestamps.
  - Audit events — decision edits, manual assignments, validation runs, cancellations, and validation decisions.
- **Acceptance criteria:**
  - [ ] Conflicting Papers are accessible through the In conflict phase-Papers filter and display assignments, decisions, and history.
  - [ ] Eligible users can edit conflicting decisions; Owner/PM can add a manual reviewer assignment.
  - [ ] Owner/PM can assign validation from the phase dashboard, selecting validators, scope, criteria, percentage, and previewed Papers.
  - [ ] Reviewers and Guests cannot receive validation assignments; all members retain read-only validation access.
  - [ ] Eligible users can validate their next assigned Paper through Validate.
  - [ ] Validation decisions apply Info, Normal, or Veto and update the Paper outcome correctly.
  - [ ] Validation assignments and decisions can be cancelled or revised without losing audit history.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Resolve conflicts and perform optional Screening validation while preserving history.

## Proposed prerequisites

- [30 — Implement Screening Assignment and Reviewer Decision Flow](30-screening.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Resolve phase overrides or project defaults; reject mutations while the phase is closed.
2. Add results/assignments/history to conflicting Paper details; Owner/Manager/Admin can edit authorized conflicting decisions, while assignees edit only their own.
3. Allow manual Add reviewer for non-Guests with Normal/Veto/Info and a note; recalculate outcomes after decisions.
4. Implement Decision/Criteria and Unanimity/Majority rules; ties remain conflicts, and an Exclude majority with different criteria remains conflicted when conflict type is Criteria.
5. Create Validation only when enabled: Assign validation, Validate, assignments, and results, with member read access according to the resolved policy.
6. Preview uses final Excluded Papers, `all_excluded` or `excluded_by_criteria` scope, percentage, and uniform sampling; distribute among Owner/Manager/Validator, excluding Reviewer/Guest and enforcing independence.
7. Keep the validated preview set fixed; return an updated preview if eligibility changes. The run's percentage does not change phase configuration.
8. Reuse Screen for the caller's validation decision with server-resolved Info/Normal/Veto type; cancelling an existing decision requires confirmation and audited revocation.
9. Persist run/configuration snapshot/idempotency, assignments, decisions, and history; conflicts are derived outcomes, not a separate table.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the extensive UI behavior/API behavior/conflict-rule fields.

- [ ] The exact conflict-detection matrix is implemented: 1 reviewer → never a normal-review conflict; conflict type Decision → any Include/Exclude split conflicts; conflict type Criteria → an Include/Exclude split **or** an all-Exclude set using different exclusion criteria both conflict; Unanimity requires full agreement (decision + criterion where applicable); Majority requires a strict, non-tied majority, and under Criteria+Majority a majority Exclude with differing criteria still leaves the Paper In conflict; a tied vote is always In conflict.
- [ ] `ScreeningPaperOutcome` remains the single source of truth for conflict state — there is no separate "conflict" table; `In conflict` is purely a derived value.
- [ ] Editing a conflicting decision is permitted for Owner/Manager/Admin on any eligible decision, and for Reviewers/Validators only on their own — a Guest never sees or can trigger an Edit action, and every case recalculates the Paper outcome immediately.
- [ ] "Add reviewer" (manual assignment) is available only to Owner/Manager/Admin while the phase is open, requires an eligible non-Guest assignee and an assignment type (Normal/Veto/Info), and is recorded with Manual mode.
- [ ] Screening history is append-only and visible read-only to every member including Guests; cancelling an assignment/decision never removes its historical entries, and a conflict-resolving edit is recorded with the "Conflict resolution edit" operation.
- [ ] The Validation navigation group and its dashboard actions appear only when Screening Validation is enabled for the phase; "Assign validation" is Owner/Manager/Admin-only, "Validate" is Owner/Manager/Validator-only, and all members (including Guests) retain read-only access to Validation pages.
- [ ] The validation-assignment preview and the final "Assign validation" confirmation use **identical** selection logic (scope, percentage, validator eligibility, independence rule) — if eligibility changes between preview and confirmation, the system returns a refreshed preview rather than silently assigning a different Paper set.
- [ ] Validation eligibility is restricted to final Excluded Papers under the chosen scope (`all_excluded` or `excluded_by_criteria`, the latter only when exclusion criteria exist), excludes Papers already assigned for validation, samples the requested percentage uniformly at random, and — when configured — excludes a user who already screened that Paper (the independence rule).
- [ ] Only Owner/Manager/Validators can receive a validation assignment; Reviewers and Guests never can, even via a crafted API call.
- [ ] Setting the assignment-run percentage does not alter the phase's stored validation configuration — it is a per-run parameter only, initialised from (but not written back to) that configuration.
- [ ] Submitting a validation decision requires an active assignment belonging to the caller, applies the server-resolved Info/Normal/Veto handling, and updates the Paper's outcome consistently with that handling.
- [ ] Cancelling a validation assignment (Owner/Manager/Admin only) preserves audit history; if a decision already exists, cancellation requires explicit confirmation, revokes the decision, and recalculates the affected outcome.
- [ ] Automated tests cover: each conflict-type/resolution-rule combination producing the expected outcome; a Reviewer's rejected attempt to edit another user's conflicting decision; the preview-vs-confirmation eligibility-drift scenario; a Reviewer/Guest rejected from receiving a validation assignment; and a validation-decision cancellation correctly reverting the Paper outcome.
- [ ] Report which commands were run and their actual results, and record the Veto/Info/Normal semantics actually implemented if not already defined elsewhere in the codebase (see Gaps).

## Gaps and pending decisions

The wizard ([18](18-protocol-wizard.md)) offers Included/Excluded/both for screening validation scope, but this task limits validation to final Excluded Papers only — implement only what this ticket specifies, per [pending decisions](PENDING-DECISIONS.md) item 9. Multiple Veto/Info/Normal semantics must be specified explicitly if the existing code/contract does not already define them precisely (e.g., does a single Veto override the phase outcome regardless of other validators' decisions?) — do not assume a behavior.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [31.01 — Resolve phase overrides or project defaults](sub-issues/31-01.md)
- [31.02 — Add results/assignments/history to conflicting Paper details](sub-issues/31-02.md)
- [31.03 — Allow manual Add reviewer for non-Guests with Normal/Veto/Info and a note](sub-issues/31-03.md)
- [31.04 — Implement Decision/Criteria and Unanimity/Majority rules](sub-issues/31-04.md)
- [31.05 — Create Validation only when enabled: Assign validation, Validate, assignments, and results, with member…](sub-issues/31-05.md)
- [31.06 — Preview uses final Excluded Papers, all_excluded or excluded_by_criteria scope, percentage, and uniform…](sub-issues/31-06.md)
- [31.07 — Keep the validated preview set fixed](sub-issues/31-07.md)
- [31.08 — Reuse Screen for the caller's validation decision with server-resolved Info/Normal/Veto type](sub-issues/31-08.md)
- [31.09 — Persist run/configuration snapshot/idempotency, assignments, decisions, and history](sub-issues/31-09.md)
