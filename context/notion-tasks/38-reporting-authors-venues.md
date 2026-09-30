# 38 — Implement Reporting Authors and Venues

## Source and usage

- Source: [Implement Reporting Authors and Venues](https://app.notion.com/p/Implement-Reporting-Authors-and-Venues-3d0499360293805ab652df6cf7e6c32b).
- Notion ID: `3d0499360293805ab652df6cf7e6c32b`.
- Source domain: Reporting & Export.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide read-only Reporting pages for Authors and Venues based only on reportable Papers. A reportable Paper is active, not excluded, and has a completed saved data-extraction/classification record. When Quality Assessment is enabled, the Paper must also remain finally Included after QA. These pages are reporting views. They do not replace the editable Authors and Venues directories in Paper library.
- **Functional Domain:** Reporting & Export
- **Behavior:** Empty
- **Users and permissions:** All project members, including Guests, can view Reporting Authors and Venues in read-only mode.
- **UI behavior:**
  - Within Reporting, provide: Authors; Venues.
  - **Authors** — searchable, sortable, paginated table: Author | Included Papers | First-author Papers. Include only authors linked to at least one reportable Paper. Included Papers = number of reportable Papers where the author appears. First-author Papers = number of reportable Papers where the author is first in the ordered author list.
  - **Venues** — searchable, sortable, paginated table: Venue | Year | Included Papers. Include only venues linked to at least one reportable Paper. Included Papers = number of reportable Papers associated with that venue and year.
  - **Shared behavior** — support search across the displayed values; support sorting by every displayed column; paginate long result sets. Do not provide add, edit, or delete actions in Reporting. Metadata edits remain available only in Paper library.
- **API behavior:** Provide read-only, project-scoped operations to: list reporting authors; list reporting venues. For both: derive results only from reportable Papers; apply search, sorting, and pagination server-side; calculate paper counts server-side; enforce project membership and reporting access.
- **Data model:** No new reporting-specific author or venue tables are required. Reuse: Paper; Author; PaperAuthor; Venue; completed Classification/Data Extraction records; QA outcome when Quality Assessment is enabled.
- **Acceptance criteria:**
  - [ ] Reporting Authors lists only authors linked to reportable Papers.
  - [ ] Reporting Venues lists only venues linked to reportable Papers.
  - [ ] Author counts distinguish all author appearances from first-author appearances.
  - [ ] Venue counts are grouped by venue and year.
  - [ ] Both pages support search, sorting, and pagination.
  - [ ] Reporting pages are read-only and project-isolated.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Report authors and venues exclusively from reportable Papers.

## Proposed prerequisites

- [25 — Implement Authors and Affiliations Directory](25-authors-affiliations.md)
- [26 — Implement Venue Directory](26-venues.md)
- [34 — Implement QA Results, Low-quality Exclusion, and Progress](34-qa-results.md)
- [35 — Implement Classification Assignment and Classification Workspace](35-classification.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Define reportable: active, non-excluded, completed saved classification/extraction; enabled QA requires a final Included outcome.
2. Create authorized, project-scoped, read-only queries with server-side search, sorting, and pagination.
3. Authors: include only reportable links and distinguish Included Papers from First-author Papers using author order.
4. Venues: group reportable Papers by venue and year.
5. Create tables without Add/Edit/Delete; reuse Author/PaperAuthor/Venue/Paper/classification/QA outcome without dedicated reporting tables.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the precisely worded "reportable Paper" definition in the Description.

- [ ] "Reportable" is applied consistently and exactly: active, not excluded, has a completed saved data-extraction/classification record, and — when Quality Assessment is enabled for the project — also has a final QA-Included outcome. A Paper failing any one of these conditions never appears in either report.
- [ ] Reporting Authors lists only authors with at least one reportable-Paper link; "Included Papers" counts every reportable-Paper appearance, "First-author Papers" counts only reportable Papers where the author is first in the ordered author list — these are two distinct, independently correct counts, not one reused for both columns.
- [ ] Reporting Venues lists only venues linked to at least one reportable Paper, grouped by (venue, year), with "Included Papers" counting reportable Papers for that exact venue+year pair.
- [ ] Both tables support search across displayed values, sorting by every displayed column, and pagination for long result sets.
- [ ] Neither page exposes any Add/Edit/Delete action — these reporting views are strictly read-only; all metadata editing remains in Paper Library ([25](25-authors-affiliations.md)/[26](26-venues.md)).
- [ ] All project members, including Guests, can view both pages read-only; results never cross project boundaries.
- [ ] No new reporting-specific database table is introduced — both pages compute from existing `Paper`, `Author`, `PaperAuthor`, `Venue`, classification/extraction, and QA-outcome records.
- [ ] Automated tests cover: an author/venue correctly excluded when its only Papers are non-reportable (pending, excluded, or QA-excluded when QA is enabled); the Included-vs-First-author count distinction for a multi-author Paper; and venue grouping across two different years for the same venue name.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

No specific contradiction is highlighted in this summary. Cross-cutting decisions in the pending-decisions register still apply. Empty source fields (Behavior) do not authorize expanding scope.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [38.01 — Define reportable: active, non-excluded, completed saved classification/extraction](sub-issues/38-01.md)
- [38.02 — Create authorized, project-scoped, read-only queries with server-side search, sorting, and pagination](sub-issues/38-02.md)
- [38.03 — Authors: include only reportable links and distinguish Included Papers from First-author Papers using…](sub-issues/38-03.md)
- [38.04 — Venues: group reportable Papers by venue and year](sub-issues/38-04.md)
- [38.05 — Create tables without Add/Edit/Delete](sub-issues/38-05.md)
