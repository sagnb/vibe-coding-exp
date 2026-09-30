# 33 — Implement Quality Assessment Assignment and Assessment Flow

## Source and usage

- Source: [Implement Quality Assessment Assignment and Assessment Flow](https://app.notion.com/p/Implement-Quality-Assessment-Assignment-and-Assessment-Flow-3b3499360293801b86c4fd3011316bb5).
- Notion ID: `3b3499360293801b86c4fd3011316bb5`.
- Source domain: Quality Assessment.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide the Quality Assessment workspace for a project where Quality Assessment is enabled. The workspace uses the active QA configuration already managed in Project settings: QA questions, shared response scale, score threshold, and optional validation settings. This ticket does not edit those settings. Each eligible Paper receives at most one active QA assignment. The assessor answers every active QA question using the shared configured response scale. The Paper score is the sum of the selected response scores.
- **Functional Domain:** Quality Assessment
- **Behavior:** Current ReLiS selects active, screening-included papers not already assigned to QA. A Project Admin selects eligible project members (review-capable, non-Guest), optionally limits the number of papers, randomizes allocation, and records an undoable operation code. Reviewers work from My Assignments, Pending, and Done lists. Selecting an answer creates/updates one `qa_result` per paper/question; the assignment becomes Done only when all active questions are answered.
- **Users and permissions:**
  - Project Owner, Project Managers, and Application Administrators can: open or close Quality Assessment; create or cancel QA assignments; view all QA assignments and assessed Papers.
  - Project Owner, Project Managers, Reviewers, and Validators can: access an open QA workspace; receive QA assignments; assess and edit only their own assigned Papers while QA is open.
  - Guests have read-only access only and cannot receive assignments or submit assessments.
- **UI behavior:**
  - **QA landing page** — displays the QA state: Open or Closed. Owner/Manager/Admin can open/close QA. When open, all project members can open the workspace. Closing preserves assignments/answers but prevents assignments and assessment changes.
  - **QA workspace** provides: Dashboard; Papers; Assess; Assignments; Assessed Papers; Results.
  - **Papers** — tabs All papers | Pending | Assessed | Excluded; a searchable/filterable/sortable/paginated table: Paper key | Title | QA status | Score | Assessed by. Title opens the Paper details page in QA context. Score displays `—` until complete. Excluded means explicitly excluded from the review after QA.
  - **Assess** — opens the connected user's next pending QA assignment. Displays: Paper key and title; abstract when available; Paper preview; every active QA question in configured order; the shared configured response scale for each question; Save and next. Rules: one response is required for every active question; the assignment becomes complete only when every question has a response; the assessor may edit only their own assessment while QA is open; empty state when no pending assignment exists.
  - **Assignments** — tabs My pending | My assignments | All assignments; columns Paper key | Title | Assigned to | Assigned by | Assigned at | State. Every project member can view; Cancel shown only to Owner/Manager/authorised Admin.
  - **Assign papers** — shown only to Owner/Manager/Admin. Displays: a multi-select of eligible project members (chips; roles shown); "Assign all eligible papers" (enabled by default); an optional number-of-Papers input when Assign all is disabled; a preview table Paper key | Title. Eligible Papers are: finally Included in the last Screening phase when Screening is enabled; otherwise active non-excluded project Papers; not already assessed or actively assigned for QA. Assignments are distributed evenly across selected members.
- **API behavior:**
  - Resolve QA configuration and permissions server-side. Reject QA mutations while QA is closed.
  - Operations: retrieve and open/close the QA workspace; list QA Papers and assignments; preview and create a QA assignment run; cancel an eligible QA assignment; retrieve the caller's next pending QA assignment; submit or update the caller's QA responses.
  - When saving an assessment: require an active assignment belonging to the caller; require one valid response for every active QA question; calculate the Paper score from response scores; mark the assignment as completed; update the QA Paper status.
- **Data model:** (stored in the Project DB)
  - `QualityAssessmentAssignmentRun` — initiating actor, selected members, requested count, timestamps.
  - `QualityAssessmentAssignment` — Paper, assignee, assigning actor, assignment run; state (Pending/Completed/Cancelled).
  - `QualityAssessmentResponse` — assignment, Paper, QA question, selected response, score.
  - `QualityAssessmentPaperOutcome` — Paper, assessment state, total score, QA exclusion state.
- **Acceptance criteria:**
  - [ ] Eligible Papers can be assigned evenly to selected non-Guest members.
  - [ ] An assessor can complete only their own QA assignment.
  - [ ] A QA assessment requires one response for every active question.
  - [ ] The Paper score equals the sum of selected response scores.
  - [ ] Closing QA blocks new assignments and assessment edits without deleting data.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Assign QA and record shared-scale responses, with one active assignment per Paper.

## Proposed prerequisites

- [21 — Implement Direct Project Settings Management](21-project-settings.md)
- [24 — Implement Project Paper List](24-paper-list.md)
- [30 — Implement Screening Assignment and Reviewer Decision Flow](30-screening.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Resolve the existing QA configuration; this task does not edit questions/scales/thresholds.
2. Create an Open/Closed workspace with Dashboard/Papers/Assess/Assignments/Assessed/Results; Guests have read-only access.
3. Owner/Manager/Admin opens/closes and creates/cancels assignments; preview includes final Screening-included Papers, or active non-excluded Papers without Screening, not already assessed/assigned.
4. Distribute evenly among non-Guest members and persist the run/assignments.
5. Implement the caller's next Paper, preview, ordered questions, and Save and next; require a valid response to every active question.
6. Calculate the score sum on the server and mark completion/outcome; only the assignee edits while QA is open.
7. Display searchable/sortable/paginated tables; incomplete scores are `—`. Closing preserves everything and blocks mutations.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the UI behavior/API behavior fields.

- [ ] QA configuration (questions, response scale, threshold, validation settings) is read from [21 — Project Settings](21-project-settings.md) only — this ticket never creates, edits, or deletes a QA question/answer/threshold.
- [ ] Opening/closing QA is restricted to Owner/Manager/Admin; closing preserves every existing assignment and response but blocks new assignments and any further edit — verified with a test that attempts a mutation while closed and gets rejected.
- [ ] Bulk assignment eligibility is exactly: finally Included in the last Screening phase when Screening is enabled, otherwise active non-excluded Papers, and in both cases not already assessed or already actively assigned for QA; eligible assignees exclude Guests; assignments are distributed evenly across selected members.
- [ ] Each eligible Paper receives **at most one active** QA assignment at a time (verified with a test that a second bulk-assignment run does not create a duplicate active assignment for an already-assigned Paper).
- [ ] The Assess flow opens only the caller's own next pending assignment, shows every active QA question in configured order with the shared response scale, and requires a response to every active question before the assignment can be marked complete.
- [ ] The Paper's QA score is calculated server-side as the sum of the selected response scores — never accepted as a client-submitted total.
- [ ] An assessor can view/edit only their own assessment, and only while QA is open — a cross-user or closed-QA submission attempt is rejected.
- [ ] The Papers table shows `—` for Score until the assessment is complete, and correctly buckets Papers into All/Pending/Assessed/Excluded tabs.
- [ ] Automated tests cover: eligible-Paper computation with and without Screening enabled; even distribution across multiple selected assessors; a rejected duplicate assignment for an already-assigned Paper; a rejected cross-user assessment edit; and score calculation correctness for a multi-question assessment.
- [ ] Report which commands were run and their actual results, and explicitly confirm that no QA-validation runtime (assign/validate/results) was implemented here — see Gaps.

## Gaps and pending decisions

The Description mentions optional QA-validation settings (configured in Settings), but [34 — QA Results, Low-quality Exclusion, and Progress](34-qa-results.md) explicitly excludes any QA-validation runtime workflow from its own scope, and no other of the 42 pages implements it either. Do not add a validation workflow to this task — implement only assignment and assessment as specified here.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [33.01 — Resolve the existing QA configuration](sub-issues/33-01.md)
- [33.02 — Create an Open/Closed workspace with Dashboard/Papers/Assess/Assignments/Assessed/Results](sub-issues/33-02.md)
- [33.03 — Owner/Manager/Admin opens/closes and creates/cancels assignments](sub-issues/33-03.md)
- [33.04 — Distribute evenly among non-Guest members and persist the run/assignments](sub-issues/33-04.md)
- [33.05 — Implement the caller's next Paper, preview, ordered questions, and Save and next](sub-issues/33-05.md)
- [33.06 — Calculate the score sum on the server and mark completion/outcome](sub-issues/33-06.md)
- [33.07 — Display searchable/sortable/paginated tables](sub-issues/33-07.md)
