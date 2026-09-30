# 36 — Implement the data extraction dashbord and progess

## Source and usage

- Source: [Implement the data extraction dashbord and progess](https://app.notion.com/p/Implement-the-data-extraction-dashbord-and-progess-3b349936029380fe8313d1b51f856f4c).
- Notion ID: `3b349936029380fe8313d1b51f856f4c`.
- Source domain: Data Extraction.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist. Note the Notion page title itself contains typos ("dashbord", "progess"); reproduced verbatim.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide a Classification dashboard with personal and project-wide completion gauges, plus a detailed Classification Progress page.
- **Functional Domain:** Data Extraction
- **Behavior:**
  - Legacy ReLiS shows My classification completion and Overall classification completion gauges. Personal progress is based on the user's active classification assignments. Overall progress is based on all non-excluded Papers eligible for Classification. A Paper is processed when it has an active Classification. Excluded Papers are removed from Classification completion counts.
- **Users and permissions:**
  - Project Owner, Project Managers, Reviewers, Validators, and Application Administrators can view Classification progress.
  - Guests have read-only access.
- **UI behavior:**
  - **Classification dashboard** — actions: "Assign papers" (opens the Classification assignment page; visible only to Owner/Manager/Admin); "Classify" (opens the caller's next pending classification assignment); "Progress" (opens the Classification Progress page); "Results" (displays as "Coming soon" — belongs to the Reporting ticket).
  - Two completion cards.
  - **My classification progress** — All | Classified | Pending | Completion gauge. All: active, non-excluded classification assignments of the connected user. Classified: the user's completed classification assignments. Pending: All − Classified. Empty state when the user has no active classification assignment.
  - **Overall classification progress** — All | Classified | Pending | Completion gauge. All: active, non-excluded Papers eligible for Classification. Classified: eligible Papers with an active Classification. Pending: All − Classified. Empty state when no Paper is eligible.
  - **Classification Progress page** — a Completion section; for each project member with active classification assignments: member name; Completed / Assigned; completion percentage; a horizontal progress bar. A final Total progress bar: `Total · Classified / Eligible Papers · Percentage` — uses the same values as Overall classification progress. Excluded Papers do not count in the total. Reassigned and cancelled assignments do not count for the former assignee. Empty state when no classification assignment exists. Use responsive stacked progress cards or bars.
- **API behavior:**
  - Use Hono as the authenticated HTTP/API layer. Provide read operations to: retrieve the Classification dashboard; retrieve detailed Classification progress.
  - Calculate metrics server-side. A Paper is eligible when it is active, non-excluded, and: Included by Quality Assessment when QA is enabled; otherwise, Included in the final Screening phase when Screening is enabled; otherwise, available in the Paper Library.
  - Exclude cancelled and reassigned assignments from active personal workload. A reassigned Paper remains pending for its new assignee until that assignee saves or confirms the Classification.
- **Data model:** (uses existing Project DB records)
  - `ClassificationAssignment` — assignee, state, assigned time, completion time.
  - `Classification` — active Classification for a Paper.
  - Paper eligibility and exclusion state.
- **Acceptance criteria:**
  - [ ] The dashboard displays accurate My and Overall Classification completion cards.
  - [ ] Excluded Papers do not appear in Classification completion totals.
  - [ ] Reassigned and cancelled assignments do not remain in the former assignee's workload.
  - [ ] The Progress page shows each assigned user and a Total row.
  - [ ] All counts are scoped to the current project.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Display personal/global classification dashboards and progress.

## Proposed prerequisites

- [35 — Implement Classification Assignment and Classification Workspace](35-classification.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Create Hono dashboard/progress reads calculated on the server per project.
2. Personal: active, non-excluded assignments, completed and pending; remove cancelled/reassigned work from the former assignee.
3. Global: active eligible non-excluded Papers, classified Papers with active records, and pending Papers; eligibility considers QA, Screening, or library.
4. Create All/Classified/Pending/gauge cards and Completed/Assigned bars by member, plus Total Classified/Eligible.
5. Reassigned work remains pending for the new assignee until saved/confirmed, according to this task's API rule.
6. Create Assign/Classify/Progress actions and Results "Coming soon" until Reporting integration; show empty states and Guest read-only access.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior field's precise eligibility and exclusion rules.

- [ ] "My classification progress" counts only the connected user's **active, non-excluded** classification assignments; Classified = the user's completed ones; Pending = All − Classified; an empty state shows when the user has zero active assignments.
- [ ] "Overall classification progress" counts all active, non-excluded Papers eligible for Classification; Classified = eligible Papers with an active Classification; Pending = All − Classified; an empty state shows when no Paper is eligible.
- [ ] Paper eligibility uses the exact precedence from API behavior: active + non-excluded, **and** (Included by QA when QA enabled) **else** (Included in the final Screening phase when Screening enabled) **else** (available in the Paper Library) — the same precedence as [35](35-classification.md), applied consistently here.
- [ ] Excluded Papers never appear in any completion total (personal or overall) — verified with a test that excluding a Paper removes it from both counts.
- [ ] Cancelled and reassigned assignments are excluded from the **former** assignee's active personal workload — their "My classification progress" must not still count a Paper they no longer own.
- [ ] A reassigned Paper remains `Pending` for the **new** assignee until that assignee actually saves/confirms the Classification — the mere pre-existence of a Classification record (created by the previous assignee) does not automatically count as "Classified" for the new assignee's personal progress (see the cross-reference to [35](35-classification.md)'s Gaps on this exact point).
- [ ] The Classification Progress page lists one row per project member with active assignments (name, Completed/Assigned, percentage, progress bar) plus a Total row reading `Total · Classified / Eligible Papers · Percentage`, using the same numbers as the Overall card — the per-user rows and the Total must never use different eligibility rules.
- [ ] Dashboard actions gate correctly: "Assign papers" only for Owner/Manager/Admin; "Classify" for anyone eligible to classify; "Progress" for all viewers (including Guests, read-only); "Results" always shows "Coming soon" in this ticket (Reporting integration is a separate ticket, [37](37-reporting-charts.md)).
- [ ] All counts and rows are scoped strictly to the current project.
- [ ] Automated tests cover: an excluded Paper disappearing from both personal and overall counts; a cancelled/reassigned assignment disappearing from the former assignee's personal progress; the reassignment "still pending for new assignee until confirmed" rule; and Total-row consistency with the Overall card.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

The reassignment rule here (a reassigned Paper stays pending for the new assignee until they save/confirm) must be read together with [35 — Classification Assignment and Classification Workspace](35-classification.md)'s own reassignment behavior, which gives the new assignee immediate access to view/edit the existing Classification — define assignment state and "completion" precisely so record-existence is never confused with the new assignee's own confirmed work, per [pending decisions](PENDING-DECISIONS.md) item 12. Personal assignment-based progress and global Paper-based progress intentionally differ (assignments vs. Papers) — do not try to reconcile their denominators into one number.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [36.01 — Create Hono dashboard/progress reads calculated on the server per project](sub-issues/36-01.md)
- [36.02 — Personal: active, non-excluded assignments, completed and pending](sub-issues/36-02.md)
- [36.03 — Global: active eligible non-excluded Papers, classified Papers with active records, and pending Papers](sub-issues/36-03.md)
- [36.04 — Create All/Classified/Pending/gauge cards and Completed/Assigned bars by member, plus Total…](sub-issues/36-04.md)
- [36.05 — Reassigned work remains pending for the new assignee until saved/confirmed, according to this task's API rule](sub-issues/36-05.md)
- [36.06 — Create Assign/Classify/Progress actions and Results Coming soon until Reporting integration](sub-issues/36-06.md)
