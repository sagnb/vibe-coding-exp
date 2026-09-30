# 39 — Implement CSV and BibTeX Reporting Exports

## Source and usage

- Source: [Implement CSV and BibTeX Reporting Exports](https://app.notion.com/p/Implement-CSV-and-BibTeX-Reporting-Exports-3b349936029380cd93b2d6447bdbbce0).
- Notion ID: `3b349936029380cd93b2d6447bdbbce0`.
- Source domain: Reporting & Export.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide direct project-scoped CSV and BibTeX exports from the Reporting area. The new relis keeps the export scopes provided by legacy ReLiS, but generates the requested file from the current Project DB and downloads it immediately. Generated files are not stored, reused, or made available through predictable server paths.
- **Functional Domain:** Reporting & Export
- **Behavior:** Legacy ReLiS provides: a CSV export of all active Papers; a CSV export of Papers excluded during Screening; a CSV export of Papers excluded during Classification/Data Extraction; a CSV export of completed Classification/Data Extraction results; BibTeX exports for all, included, and excluded Papers. The new relis keeps these export scopes and uses canonical current workflow data instead of legacy status-field inconsistencies.
- **Users and permissions:** Every application user can generate and download project exports. *(This is directly contradicted by the API's own project/export-authorization requirement — see Gaps.)*
- **UI behavior:**
  - Within Reporting > Exports, display one card for each export: Papers CSV (all active project Papers; includes Paper key, title, DOI/URL, preview, abstract, and year); Screening phase exclusions CSV (Papers finally excluded during Screening; includes Paper metadata, excluding actor, exclusion criterion, and note when available); Classification exclusions CSV (Papers excluded during Classification/Data Extraction; includes Paper metadata, excluding actor, criterion, and note when available); Classification/Data Extraction results CSV (Papers with a saved active Classification/Data Extraction record; includes Paper metadata and saved extraction values); All Papers BibTeX (all active Papers with stored BibTeX content); Included Papers BibTeX (Papers with a current final Included outcome); Excluded Papers BibTeX (Papers with a current final Excluded outcome).
  - Each card displays a short description of its scope, the current number of matching Papers, and a "Generate and download" action. On selection: show a blocking generation state; download the file when generation succeeds; display a safe error message when generation fails.
- **API behavior:**
  - Enforce current project context and export permission. Provide one operation to generate and stream each allowed CSV or BibTeX export.
  - For every export: resolve the export scope server-side; query only the current Project DB; use the fixed fields defined for that export type; generate UTF-8 CSV with correct escaping for separators, quotes, and line breaks; return the generated file directly to the browser; never accept a filesystem path or filename from the client; do not persist the generated file, its content, or a download URL; write an audit event containing export type, actor, timestamp, and exported Paper count.
- **Data model:** No generated-file, export-job, worker-job, retention, or artifact tables are required. Reuse existing Project DB records for: Papers; Screening outcomes and exclusion data; Classification/Data Extraction records and exclusions; stored BibTeX content. Store a lightweight export audit event only.
- **Acceptance criteria:**
  - [ ] Authorised users can directly download each supported CSV and BibTeX export.
  - [ ] Papers CSV contains all active project Papers.
  - [ ] Classification/Data Extraction CSV contains only Papers with saved active extraction data.
  - [ ] Included and Excluded BibTeX exports use the current final Paper outcome.
  - [ ] Exclusion CSV exports include available exclusion details.
  - [ ] Every export is isolated to the current project.
  - [ ] Generated files are downloaded immediately and are not retained on the server.
  - [ ] No worker, export history, retry, expiry, revoke, or path-based download flow is implemented.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Generate and stream CSV/BibTeX directly without storing exported files.

## Proposed prerequisites

- [24 — Implement Project Paper List](24-paper-list.md)
- [30 — Implement Screening Assignment and Reviewer Decision Flow](30-screening.md)
- [35 — Implement Classification Assignment and Classification Workspace](35-classification.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Define operations by export type, authorized within the current project; reject client-supplied paths or filenames.
2. Create cards for Papers CSV; Screening exclusions CSV; Classification exclusions CSV; Classification/Data Extraction results CSV; and All/Included/Excluded BibTeX.
3. Resolve scope on the server: active Papers; final Screening exclusions; Classification exclusions; active saved classifications; stored BibTeX and current outcomes.
4. CSV includes the specified fixed fields: key/title/DOI/URL/preview/abstract/year and, depending on the export, exclusion actor/criterion/note or extracted values.
5. Generate UTF-8 with separator/quote/newline escaping; stream directly and display generation status/safe errors on the client.
6. Audit type, actor, timestamp, and count; create no job, artifact history, expiry/revoke flow, or persisted URL.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/UI behavior fields — note the Users-and-permissions contradiction flagged in Gaps.

- [ ] Every export endpoint enforces current project context and an actual export permission check server-side — "every application user" from Users and permissions is not implemented literally; only authorised (project-scoped) users can trigger a generation (see Gaps for how this was resolved).
- [ ] Each of the seven export cards (Papers CSV; Screening exclusions CSV; Classification exclusions CSV; Classification/Data Extraction results CSV; All/Included/Excluded BibTeX) resolves its scope entirely server-side from the current Project DB — the client supplies no filter, path, or filename.
- [ ] Papers CSV contains exactly all active project Papers with key, title, DOI/URL, preview, abstract, and year.
- [ ] Screening/Classification exclusion CSVs contain only Papers finally excluded by that respective workflow, including the excluding actor, criterion, and note when available.
- [ ] Classification/Data Extraction results CSV contains only Papers with a saved **active** extraction/classification record, including the saved extraction values.
- [ ] Included/Excluded BibTeX exports use the Paper's **current final** outcome (not a stale/cached one) — a Paper whose outcome has since changed reflects the new outcome on the next export.
- [ ] Generated CSV is valid UTF-8 with correct escaping for the configured separator, quotes, and embedded line breaks (verify with a fixture containing a comma/quote/newline inside a field).
- [ ] The file streams directly to the browser as the HTTP response — it is never written to disk, object storage, or any persisted location, and no download URL is ever generated or returned.
- [ ] Every generation writes a lightweight audit event with export type, actor, timestamp, and exported Paper count — nothing more (no job record, no retry/expiry/revoke state).
- [ ] The UI shows a blocking generation state during the request and a safe, non-technical error message on failure (never a raw stack trace or DB error).
- [ ] Automated tests cover: each of the seven scopes returning the exact expected Paper set against a seeded fixture; CSV escaping correctness; and a rejected generation attempt for an unauthorised caller.
- [ ] Report which commands were run and their actual results, and explicitly record the authorization policy actually implemented for "every application user" per the decision in Gaps.

## Gaps and pending decisions

Users and permissions says every application user can generate exports, but the API behavior requires project/export authorization — a direct contradiction on this card. Define whether this covers all project members, Guests, or an authorized public audience (e.g., for a Published project) before implementing, per [pending decisions](PENDING-DECISIONS.md) item 14. This task's "no job/no persistence" contract also conflicts with the generic jobs/storage platform's expectation of persisted exports ([06](06-job-platform.md)/[07](07-storage.md)) — follow this ticket's specific direct-streaming contract, not the generic one, once the decision above is recorded.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [39.01 — Define operations by export type, authorized within the current project](sub-issues/39-01.md)
- [39.02 — Create cards for Papers CSV](sub-issues/39-02.md)
- [39.03 — Resolve scope on the server: active Papers](sub-issues/39-03.md)
- [39.04 — CSV includes the specified fixed fields: key/title/DOI/URL/preview/abstract/year and, depending on the…](sub-issues/39-04.md)
- [39.05 — Generate UTF-8 with separator/quote/newline escaping](sub-issues/39-05.md)
- [39.06 — Audit type, actor, timestamp, and count](sub-issues/39-06.md)
