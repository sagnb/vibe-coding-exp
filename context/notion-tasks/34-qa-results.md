# 34 — Implement QA Results, Low-quality Exclusion, and Progress

## Source and usage

- Source: [Implement QA Results, Low-quality Exclusion, and Progress](https://app.notion.com/p/Implement-QA-Results-Low-quality-Exclusion-and-Progress-3b3499360293807cb062eb1fa66195b3).
- Notion ID: `3b3499360293807cb062eb1fa66195b3`.
- Source domain: Quality Assessment.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist. This card has a real internal contradiction — read Gaps before implementing individual exclusion/restoration.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide the Quality Assessment dashboard, progress, and results pages. The feature displays completed QA assessments and their score relative to the configured cut-off score. A Paper is not automatically excluded because its score is below the cut-off. Project Owners and Project Managers explicitly decide whether to exclude low-quality Papers, individually or in bulk.
- **Functional Domain:** Quality Assessment
- **Behavior:** Current ReLiS sorts QA result lists by score, displays the cut-off, and flags scores below it. A Project Admin can exclude one low-quality paper (`Excluded_QA`) or confirm bulk exclusion of all papers below the cut-off; per-paper exclusion can be cancelled, returning the paper to Included/To classify *(note: this legacy per-paper exclude/restore behavior is explicitly prohibited by this card's own Acceptance criteria — see Gaps)*. QA validation selects a random percentage of completed QA papers not already assigned to validation, distributes them to eligible users, and stores Correct or Not Correct, a note, and timestamp *(note: this legacy QA-validation behavior is also explicitly out of scope per this card's own Acceptance criteria — see Gaps)*. The QA dashboard shows personal/overall QA and validation completion gauges and per-user counts.
- **Users and permissions:**
  - All project members, including Guests, can view the QA dashboard, progress, and results in read-only mode.
  - Project Owner, Project Managers, and Application Administrators can: exclude eligible low-quality Papers; restore an eligible QA-excluded Paper *(this restore capability, worded here, conflicts with the Acceptance criteria's "no individual QA exclusion, restoration" — see Gaps)*.
- **UI behavior:**
  - **QA dashboard** — actions: Assign papers; Assess; Progress; Results. Cards: My QA progress (All assignments, Assessed, Pending, completion gauge); Overall QA progress (same fields); QA result summary (Assessed Papers, Below cut-off, Excluded by QA).
  - **QA Progress page** — a Completion section; for each member with active QA assignments: user name, Completed / Assigned, completion percentage, horizontal progress bar; a final Total progress bar.
  - **QA Results page** — displays the configured global QA cut-off score at the top. Shows "Exclude assessed Papers below cut-off" only to Owner/Manager/Admin. A searchable/filterable/sortable/paginated table: Paper key | Title | Score | Assessed. Title opens the existing QA assessment page for that Paper (which shows questions and, when assessed, the selected response/score per question); read-only for users not allowed to edit that assessment. Score shows the total when complete, otherwise `—`. Assessed shows a checked indicator when every active question has a saved response, otherwise nothing. Filters: assessed/not assessed; score relative to cut-off; assessor.
  - **Low-quality exclusion** — "Exclude papers below cut-off" shown only to Owner/Manager/Admin. Before confirmation, displays: number of affected Papers; the Paper list; the active cut-off score; a clear explanation that excluded Papers cannot continue to downstream workflow steps. On confirmation, excludes only completed, non-excluded Papers whose score is below the current cut-off. For an eligible individually excluded Paper, shows "Restore QA exclusion" to the same authorised roles *(again, worded as individual exclusion/restore — see the direct conflict noted in Gaps)*.
- **API behavior:**
  - Provide read operations for: QA dashboard; QA progress; QA results; low-quality exclusion preview. Reuse the existing QA assessment read model from [33 — Assignment and Assessment](33-qa-assessment.md) when a user opens a Paper title from QA Results — it returns the Paper's questions, saved responses, and scores in read-only mode when the caller cannot edit the assessment.
  - Provide one authorised mutation: confirm a bulk low-quality exclusion. Rules: calculate the QA score from recorded response scores; evaluate the cut-off status using the current configured QA cut-off score; the bulk confirmation must use the same validated Paper set shown in its preview; exclude only completed, non-excluded Papers below the cut-off; preserve the original screening outcome when applying a QA exclusion; record the bulk exclusion and every affected Paper in the audit history.
- **Data model:** (uses existing `QualityAssessmentAssignment`, `QualityAssessmentResponse`, and `QualityAssessmentPaperOutcome` records)
  - `QualityAssessmentPaperOutcome` stores: assessment state; total QA score; cut-off status; QA exclusion state.
  - Add `QualityAssessmentExclusionRun` — initiating actor; cut-off score snapshot; affected Paper count; confirmation timestamp.
  - Add `QualityAssessmentExclusionItem` — exclusion-run reference; Paper reference; exclusion timestamp.
- **Acceptance criteria:**
  - [ ] The QA dashboard and Progress page show accurate personal and overall completion.
  - [ ] QA Results displays: Paper key | Title | Score | Assessed.
  - [ ] The configured global cut-off score is visible above the results table.
  - [ ] A completed assessment shows a checked indicator; an incomplete assessment shows no indicator.
  - [ ] Opening a Paper title displays its QA questions, saved responses, and scores.
  - [ ] QA Results supports search, filtering, sorting, and pagination.
  - [ ] Authorised users can preview and confirm one bulk exclusion of completed Papers below the cut-off.
  - [ ] The confirmed exclusion affects only the Papers displayed in the preview.
  - [ ] **No individual QA exclusion, restoration, or QA validation workflow is implemented.**
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Display QA progress/results and bulk-exclude assessed Papers below the cut-off after confirmation.

## Proposed prerequisites

- [33 — Implement Quality Assessment Assignment and Assessment Flow](33-qa-assessment.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Reuse assignments/responses/outcomes; calculate score, cut-off status, and personal/global progress on the server.
2. Create cards and member/Total bars, a Paper key | Title | Score | Assessed table, search/filter/sorting/pagination, and a visible cut-off.
3. Titles open existing assessments, read-only where required; incomplete scores are `—` and the indicator appears only for completed assessments.
4. Implement a preview of completed, non-excluded Papers below the current cut-off; show impact and the exact set.
5. Owner/Manager/Admin confirms that same validated set; preserve Screening outcomes and record run/item/cut-off snapshot/actor/timestamp.
6. Do not automatically exclude by score; document the pending decision before implementing individual exclusion/restoration.

## Definition of Done — completion checklist

**This card's own Acceptance criteria property (the final, authoritative field, reproduced above) explicitly prohibits individual QA exclusion, restoration, and any QA-validation workflow — even though the Description/Behavior/UI behavior/Users-and-permissions fields on the same card describe exactly those features.** Per this repository's practice of treating the Acceptance criteria property as authoritative when a direct conflict exists on one card, the checklist below implements only the bulk-exclusion core and explicitly excludes the conflicting individual-exclusion/restore/validation features. Do not implement the excluded items without first obtaining a decision — see Gaps.

- [ ] The QA dashboard's My/Overall QA progress cards and the dedicated Progress page compute completion accurately (Completed/Assigned per user, a Total row) purely from existing `QualityAssessmentAssignment`/`QualityAssessmentResponse` records — no new assignment is created by this ticket.
- [ ] QA Results shows Paper key | Title | Score | Assessed with the configured global cut-off score visible above the table; Score shows `—` until the assessment is complete; the Assessed indicator (checked) appears only when every active QA question has a saved response.
- [ ] Search, filtering (assessed/not-assessed, score-vs-cutoff, assessor), sorting, and pagination all work on the Results table.
- [ ] Opening a Paper title reuses [33](33-qa-assessment.md)'s read model to show its questions, saved responses, and score — rendered read-only for a caller not authorised to edit that assessment.
- [ ] Bulk exclusion: the preview and the confirmed exclusion use the **same** validated Paper set (completed, non-excluded, below the current cut-off) — the confirmed mutation must never exclude a Paper that wasn't shown in the preview the user approved.
- [ ] Confirming a bulk exclusion preserves the Paper's original Screening outcome (it does not overwrite or delete it) and records a `QualityAssessmentExclusionRun` plus one `QualityAssessmentExclusionItem` per affected Paper, with the cut-off snapshot, actor, and timestamp, and writes an audit event.
- [ ] A Paper's score falling below the cut-off never triggers automatic exclusion — exclusion happens only through the explicit, confirmed bulk action.
- [ ] **Do not implement**: an individual per-paper "Exclude"/"Restore QA exclusion" action, or any QA-validation assignment/decision workflow (Correct/Not Correct, validator distribution) — these appear in the Description/Behavior/UI fields but are explicitly excluded by the card's own Acceptance criteria.
- [ ] Automated tests cover: accurate progress-card computation; preview-set = confirmed-set equality for bulk exclusion; original Screening outcome preserved after a QA exclusion; and the Assessed-indicator/Score-placeholder logic for an incomplete assessment.
- [ ] Report which commands were run and their actual results, and explicitly record that individual exclusion/restoration and QA validation were intentionally left unimplemented pending the decision in Gaps — do not describe the feature as "complete" without that caveat.

## Gaps and pending decisions

Direct, verbatim conflict on this card: the Description, Behavior, UI behavior, and Users-and-permissions fields describe individual per-paper exclusion and restoration (and QA validation as a runtime feature), while the final Acceptance criteria property states "No individual QA exclusion, restoration, or QA validation workflow is implemented." This guide's checklist covers only the uncontested bulk-exclusion core; do not implement the conflicting individual-exclusion/restoration/validation features, and do not claim the full card is satisfied, without an explicit decision recorded in [pending decisions](PENDING-DECISIONS.md) item 10.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [34.01 — Reuse assignments/responses/outcomes](sub-issues/34-01.md)
- [34.02 — Create cards and member/Total bars, a Paper key | Title | Score | Assessed table,…](sub-issues/34-02.md)
- [34.03 — Titles open existing assessments, read-only where required](sub-issues/34-03.md)
- [34.04 — Implement a preview of completed, non-excluded Papers below the current cut-off](sub-issues/34-04.md)
- [34.05 — Owner/Manager/Admin confirms that same validated set](sub-issues/34-05.md)
- [34.06 — Do not automatically exclude by score](sub-issues/34-06.md)
