# 28 — Provide one CSV Import page organised into progressive sections.

## Source and usage

- Source: [Provide one CSV Import page organised into progressive sections.](https://app.notion.com/p/Provide-one-CSV-Import-page-organised-into-progressive-sections-3b3499360293805c916ac3d84dc191ec).
- Notion ID: `3b3499360293805c916ac3d84dc191ec`.
- Source domain: Paper Library & Import.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide an asynchronous CSV import workflow that lets authorised users configure, validate, and start a CSV import from a previously uploaded `ImportBatch`.
- **Functional Domain:** Paper Library & Import
- **Behavior:** Legacy ReLiS supports: CSV separator configuration; header-row and first-data-row selection; CSV preview; paper-field mapping; optional Paper Source and Search Strategy assignment; automatic paper-key generation when no citation key is supplied; cancellation and clear feedback for invalid input. NeoReLiS keeps these research-workflow capabilities while using a native structured wizard and an auditable import result.
- **Users and permissions:**
  - The Project Owner and Project Managers can start, validate, commit, and roll back CSV imports for their current project.
  - Reviewers, Validators, and Guests cannot start, commit, or roll back imports.
  - All requests and resulting Paper records are strictly scoped to the current project.
- **UI behavior:**
  - Provide one CSV Import page organised into progressive sections. From Paper library > Import papers, the user uploads a CSV file and creates an `ImportBatch`. Once the batch reaches `Queued`, open its CSV Import page. The page works with the batch's temporary stored file.
  - **1. CSV options** — read the batch's temporary CSV file and detect likely separator, encoding, and header row; display detected values; allow changing separator/encoding (when the preview shows incorrectly decoded characters) and specifying whether the file has a header row and which row is the first data row.
  - **2. Preview and column mapping** — display a limited preview of parsed rows; map CSV columns to Paper fields; Title mapping is required; optional mappings: Paper key, DOI, URL, Year, Abstract, BibTeX, Keywords, Authors, Venue; unused columns may be ignored. When a row has no Paper key value, generate a project-local key using the configured optional prefix and a backend-managed sequence.
  - **3. Optional import metadata** — when active Paper Sources and Search Strategies exist in project settings, display independent optional selectors for Paper Source and Search Strategy; apply selected values to every Paper created by this import.
  - **4. Validation** — validate the temporary file using the selected options/mappings; display number of valid rows, number of invalid rows, and validation errors with affected row numbers. When rows are invalid, provide: Edit mapping (focuses relevant controls/highlights affected fields); Choose another file (cancels the current uncommitted `ImportBatch`, deletes its temporary file, returns to Start a new import); Ignore invalid rows and continue (imports only valid rows). Before Save, clearly state how many Papers will be created and how many invalid rows will be ignored.
  - **Save and cancellation** — Save submits CSV options, mapping, optional metadata, and the explicit ignored-row choice for the current `ImportBatch`; after Save, show an Importing state and allow leaving the page safely while the import continues in the background. Cancel cancels the eligible `ImportBatch`, deletes its temporary file, and returns to Import papers. Refreshing/reopening retrieves the current server-side `ImportBatch` state.
  - **Import result** — display batch state (`Queued`, `Validating`, `Ready to import`, `Importing`, `Completed`, `Failed`, or `Cancelled`). When completed: created Paper count; ignored invalid-row count; safe warnings/errors when applicable; a link to Paper library > Papers. When failed: a clear, non-technical error summary and any action the server makes available.
- **API behavior:**
  - Allow CSV import only for the Project Owner, Project Managers, and authorised Application Administrators in the current project. Use the existing CSV `ImportBatch` created by the secure upload workflow ([27](27-import-intake.md)); do not accept or create another uploaded file in this ticket.
  - Provide operations to: retrieve a CSV `ImportBatch` and its current state; inspect the batch's temporary stored CSV file and return detected separator/encoding/header info and a limited preview; validate the stored CSV file with selected options/mapping; start an asynchronous CSV import for a validated batch; retrieve the current status/final result of a CSV import; cancel an eligible uncommitted batch.
  - For preview/validation: retrieve the temporary stored CSV file through the authorised `ImportBatch`; receive only the selected CSV options and mapping from the browser; return detected options, preview rows, and row-level validation errors.
  - For Save: receive the selected CSV options, mapping, optional Paper Source/Search Strategy IDs, and explicit ignored-invalid-rows choice; require a Title mapping; persist this configuration on the existing `ImportBatch`; create one idempotent CSV import job for the batch; return the current batch state without waiting for Paper creation.
  - The CSV import worker must: reparse and revalidate the temporary stored CSV file; create Papers only from valid rows; ignore invalid rows only when explicitly confirmed; generate a project-local Paper key when a row has no key value; apply the selected Paper Source/Search Strategy to every created Paper; persist the result counts and safe error summary on the `ImportBatch`; create the Papers and final Project DB batch result atomically; delete the temporary stored file after successful completion.
  - Cancellation: available only before the batch enters the commit workflow; cancels the eligible batch and deletes its temporary stored file. Return only client-safe status, progress, result counts, and error summaries.
- **Data model:**
  - When the user starts the CSV import, update the existing `ImportBatch` with: CSV options snapshot; mapping snapshot; selected Paper Source/Search Strategy references when provided; explicit ignored-invalid-rows choice; validation/import/completion/failure/cancellation timestamps; current and final result status; total-row, valid-row, invalid-row, created-Paper, and ignored-row counts; safe result summary; idempotent CSV commit-job reference.
  - Store `ImportBatchIgnoredRow` records only for invalid rows explicitly ignored during the completed import: batch reference; row number; safe validation reason.
- **Acceptance criteria:**
  - [ ] A Project Owner, Project Manager, or authorised Application Administrator can open and configure a queued CSV `ImportBatch` for the current project.
  - [ ] The CSV Import page displays a preview before any Paper is created.
  - [ ] The CSV Import page requires a Title mapping and reports row-level validation errors before Save.
  - [ ] The user can explicitly choose to ignore invalid rows and import valid rows only.
  - [ ] Optional active Paper Source and Search Strategy selections are applied consistently to every created Paper.
  - [ ] A missing Paper key is generated by the backend using the project configuration.
  - [ ] Save starts one asynchronous CSV import job for the existing `ImportBatch`.
  - [ ] The worker reparses and revalidates the stored CSV file before creating Papers.
  - [ ] The completed `ImportBatch` contains created-Paper, valid-row, invalid-row, and ignored-row counts.
  - [ ] Repeating the same Save action does not create another CSV import job, `ImportBatch`, or set of Papers.
  - [ ] Cancelling an eligible batch prevents Paper creation, deletes its temporary stored file, and marks the batch as `Cancelled`.
  - [ ] Refreshing or leaving the page does not cancel the `ImportBatch`. The user can reopen it while its temporary file remains available.
  - [ ] The original CSV is stored privately only while required by the active import workflow.
  - [ ] The original CSV is deleted immediately after successful import, cancellation, or non-retryable failure. A storage-level expiry removes any file left by an interrupted worker.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Configure, validate, and asynchronously import CSV from an existing uploaded ImportBatch.

## Proposed prerequisites

- [21 — Implement Direct Project Settings Management](21-project-settings.md)
- [24 — Implement Project Paper List](24-paper-list.md)
- [27 — Implement Secure Import Storage and Project Job Progress](27-import-intake.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Open a Queued batch and inspect its existing temporary file; detect separator, encoding, and headers and allow adjustments/first data row selection.
2. Create a limited preview and mapping: Title required; optional key, DOI, URL, year, abstract, BibTeX, keywords, authors, and venue.
3. Offer independent optional active Source and Strategy selectors; apply them to the entire batch.
4. Validate rows and display numbered errors. Allow mapping corrections, cancellation/file replacement, or explicitly confirmed ignoring of invalid rows.
5. Save persists option/mapping/metadata/choice snapshots and creates one idempotent job.
6. The worker reparses/revalidates, generates missing keys using a prefix/atomic counter, and atomically creates valid Papers plus the final batch result in the Project DB.
7. Persist total/valid/invalid/created/ignored counts and `ImportBatchIgnoredRow` only for invalid rows actually ignored.
8. Allow cancellation before commit; delete originals after success/cancellation/non-retryable failure and respect TTL. Display re-queryable states/results.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary, unusually complete source for this ticket; the items below restate it as concrete, testable points with added implementation detail from the UI/API behavior fields.

- [ ] Only a Project Owner, Project Manager, or authorised Application Administrator can open/configure a queued CSV `ImportBatch`; Reviewers/Validators/Guests are rejected both in the UI and at the API.
- [ ] CSV options (separator, encoding, header presence, first-data-row) are auto-detected and displayed, and remain user-adjustable when detection is wrong (e.g., garbled preview text signals a wrong encoding).
- [ ] The preview and column-mapping step never creates a Paper — Title mapping is mandatory before proceeding, and every other mapping (Paper key, DOI, URL, Year, Abstract, BibTeX, Keywords, Authors, Venue) is optional.
- [ ] When active Paper Sources/Search Strategies exist for the project, both selectors are independent and optional, and whatever is selected is applied to **every** Paper created by this specific import batch.
- [ ] Validation reports exact valid/invalid row counts with row-level error messages, and the user can: fix the mapping and revalidate; cancel and choose another file (which cancels the current `ImportBatch` and deletes its temp file); or explicitly confirm "ignore invalid rows and continue."
- [ ] The pre-Save summary clearly states how many Papers will be created and how many rows will be ignored — a user must see this number before committing.
- [ ] Save is idempotent: submitting Save twice (e.g., double-click, retried request) does not create a second import job, a second `ImportBatch`, or duplicate Papers.
- [ ] The import worker reparses and revalidates the stored file itself (never trusting only the browser's earlier validation pass), creates Papers only from currently-valid rows, and only ignores rows the user explicitly chose to ignore.
- [ ] A missing Paper key is generated backend-side from the configured optional prefix plus an atomic/sequence-based counter — never a client-supplied or ad hoc value.
- [ ] Paper creation and the final `ImportBatch` result (counts, status) are written atomically in the Project DB — a partial run never leaves some Papers created without the batch reflecting them, or vice versa.
- [ ] `ImportBatchIgnoredRow` records exist only for rows actually ignored in a **completed** import — not for every invalid row encountered during validation attempts.
- [ ] Cancelling an eligible (not-yet-committing) batch prevents any Paper creation, deletes the temporary file immediately, and marks the batch `Cancelled`; merely refreshing or navigating away never cancels it.
- [ ] The original CSV file is deleted immediately after successful import, cancellation, or non-retryable failure, with a storage-level expiry (per [27](27-import-intake.md)) as the safety net for an interrupted worker.
- [ ] Automated tests cover: the full happy path (detect → map → validate → save → completed with correct counts); a validation failure blocking Save until Title is mapped; explicit ignore-invalid-rows producing the expected `ImportBatchIgnoredRow` set; a duplicate Save not duplicating Papers; and a cancel deleting the temp file and blocking further Paper creation.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

Users and permissions mentions rollback ("start, validate, commit, and roll back"), but this flow's UI/API behavior provides no rollback contract or endpoint. Duplicate detection/resolution is also unspecified on this page; do not invent either feature — record them as explicit gaps per [pending decisions](PENDING-DECISIONS.md) item 15.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [28.01 — Open a Queued batch and inspect its existing temporary file](sub-issues/28-01.md)
- [28.02 — Create a limited preview and mapping: Title required](sub-issues/28-02.md)
- [28.03 — Offer independent optional active Source and Strategy selectors](sub-issues/28-03.md)
- [28.04 — Validate rows and display numbered errors](sub-issues/28-04.md)
- [28.05 — Save persists option/mapping/metadata/choice snapshots and creates one idempotent job](sub-issues/28-05.md)
- [28.06 — The worker reparses/revalidates, generates missing keys using a prefix/atomic counter, and atomically…](sub-issues/28-06.md)
- [28.07 — Persist total/valid/invalid/created/ignored counts and ImportBatchIgnoredRow only for invalid rows…](sub-issues/28-07.md)
- [28.08 — Allow cancellation before commit](sub-issues/28-08.md)
