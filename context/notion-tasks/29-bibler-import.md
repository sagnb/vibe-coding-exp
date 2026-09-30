# 29 — Implement BibTeX and EndNote Import Parsers

## Source and usage

- Source: [Implement BibTeX and EndNote Import Parsers](https://app.notion.com/p/Implement-BibTeX-and-EndNote-Import-Parsers-3b349936029380eda9f8df8b2e109280).
- Notion ID: `3b349936029380eda9f8df8b2e109280`.
- Source domain: Paper Library & Import.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Use the existing private BiBler service to preserve the legacy ReLiS BibTeX and EndNote import contract. The workflow consumes an existing queued `ImportBatch`, delegates parsing and bibliographic normalisation to BiBler, presents the parsed results for review, then commits approved valid records asynchronously.
- **Functional Domain:** Paper Library & Import
- **Behavior:** Legacy ReLiS delegates BibTeX/EndNote conversion to Bibler. The new relis preserves this.
- **Users and permissions:** Project Admins and Project Managers can initiate, review, commit, cancel, and roll back imports in their own project; Application Administrators can support any project. Parser input is never executed as code or trusted configuration.
- **UI behavior:**
  - Reuse the shared import workflow. Preview normalized bibliographic records, authors, venue, DOI, year, abstract, URLs, and unsupported-field warnings before commit.
  - When the active protocol defines Paper Sources and/or Search Strategies, display the corresponding selectors after parsing and before duplicate review. The selected values apply to every paper committed from that import batch. They are project-defined provenance reference lists; the parser does not execute a search strategy.
  - Show partial failure details without exposing raw parser stack traces.
- **API behavior:**
  - Use a private BiBler client, not a native parser adapter. The worker retrieves the temporary file from an existing queued `ImportBatch` and sends its content to the internal BiBler service using the legacy ReLiS-compatible import contract.
  - Support: BibTeX imports; EndNote exports compatible with the legacy BiBler flow.
  - Validate BiBler responses against a fixed normalized-reference contract before using them. Persist normalized import candidates, their display order, and safe per-record warnings/errors on the existing `ImportBatch`. Use the existing batch workflow for review and asynchronous commit — do not create a second persistence path.
  - Do not implement duplicate detection, duplicate resolution, or rollback in this ticket. Treat uploaded files and BiBler responses as untrusted input. Apply request timeouts, response-size limits, and bounded worker retries. Return only safe, human-readable parser errors; never expose BiBler internals, raw responses, or stack traces.
- **Data model:**
  - Project DB — reuse the shared `ImportBatch` created by the secure intake workflow ([27](27-import-intake.md)). Store normalized import candidates returned by BiBler before Paper creation: each candidate stores its display order, normalized reference fields, and safe per-record warning/error details. Candidates are not Paper records — Papers are created only after the user-approved commit workflow. Store the batch parsing, review, commit, completion, failure, and cancellation states and timestamps. Do not store raw BiBler responses, parser logs, duplicate candidates, or rollback state.
  - Private temporary file storage — store the original uploaded file only while the `ImportBatch` requires it; do not store normalized intermediate artifacts for debugging/repeatability; delete the file immediately after successful final import, cancellation, or non-retryable failure; apply the short storage-level expiry as a safety net.
  - Control DB — worker-job tracking, bounded retry attempts, and technical audit references only; each job references the internal `projectId` and `importBatchId`.
- **Acceptance criteria:**
  - [ ] BibTeX imports preserve supported bibliographic metadata, author order, venue information, DOI, and URLs.
  - [ ] EndNote imports report unsupported fields and partial failures without failing unrelated valid records.
  - [ ] Both formats use the same normalized record, duplicate, batch, and rollback workflow as CSV.
  - [ ] Parser inputs cannot execute code, exhaust resources, or expose internal stack traces.
  - [ ] For each approved legacy fixture, the import produces the expected record count and preserves the supported title, ordered authors, venue, year, DOI, abstract, URL, and citation-key values. Invalid records produce per-record warnings without preventing unrelated valid records from proceeding.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Import BibTeX/EndNote through the private BiBler service using the legacy-compatible contract.

## Proposed prerequisites

- [06 — Complete job queue and progress platform](06-job-platform.md)
- [24 — Implement Project Paper List](24-paper-list.md)
- [27 — Implement Secure Import Storage and Project Job Progress](27-import-intake.md)
- [28 — Provide one CSV Import page organised into progressive sections.](28-csv-import.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Confirm BiBler access, documentation, fixtures, and legacy contract; do not replace it with a native parser.
2. The worker reads a Queued batch's temporary file and sends it to the internal service with timeouts, response limits, and bounded retries.
3. Validate responses against a fixed normalized-reference contract; preserve ordered authors, venue, year, DOI, abstract, URL, and citation key.
4. Persist ordered candidates and safe warnings/errors on the existing batch without creating Papers yet.
5. Create a preview and Source/Strategy selection before subsequent review when configured; no strategy executes an external search.
6. Integrate user-approved asynchronous commit through the shared workflow; delete the file after success/cancellation/terminal failure.
7. Do not persist raw responses/parser logs or intermediate debugging artifacts.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points, flag the two internal contradictions in the source, and add detail grounded in the API behavior/Data model fields.

- [ ] BiBler is treated strictly as an external, untrusted service: requests use bounded timeouts, response-size limits, and a bounded worker retry policy; its raw responses, internals, and stack traces are never surfaced to the client or persisted — only safe, human-readable per-record warnings/errors.
- [ ] A response from BiBler is validated against a fixed normalized-reference contract **before** being used for anything — an unexpected/malformed response is treated as a parser failure, not partially trusted.
- [ ] Parsing produces ordered normalized candidates (title, ordered authors, venue, year, DOI, abstract, URL, citation key, plus per-record warnings) persisted on the existing `ImportBatch` — no Paper record is created at this stage.
- [ ] When the project defines active Paper Sources and/or Search Strategies, both selectors appear after parsing and before commit, and whatever is selected applies to every Paper eventually committed from this batch; no search strategy is ever executed against an external system.
- [ ] The worker retrieves the temporary file from the existing intake `ImportBatch` ([27](27-import-intake.md)) — this ticket does not accept or create a second upload path.
- [ ] A partial failure (some records invalid/unsupported) reports per-record warnings and still allows the unrelated valid records to proceed to commit — one bad record must not abort the whole batch.
- [ ] Committing approved valid records reuses the same asynchronous commit workflow as CSV ([28](28-csv-import.md)) rather than a second, parallel persistence path.
- [ ] The original uploaded file, and any BiBler raw response or parser log, are never retained beyond what [27](27-import-intake.md)'s lifecycle already allows — no additional "debugging artifact" storage is introduced.
- [ ] Automated tests run against each approved legacy fixture (BibTeX and EndNote) and verify: the expected record count; preserved title/ordered authors/venue/year/DOI/abstract/URL/citation-key values; and that an invalid record in the fixture produces a warning without blocking the other valid records.
- [ ] Report which commands were run and their actual results, and explicitly flag as unresolved (not silently implemented) the two contradictions in Gaps below: duplicate detection/rollback for BibTeX/EndNote, and this ticket's access relative to BiBler's private-service documentation/fixtures.

## Gaps and pending decisions

The API behavior excludes duplicate detection/resolution/rollback implementation, but Acceptance criteria require sharing those flows with CSV (which itself, per [28](28-csv-import.md), also has no rollback contract despite its own Users and permissions field mentioning "roll back"). Record the missing dependency without implementing a parallel path or marking the gap complete — see [pending decisions](PENDING-DECISIONS.md) item 15. Confirm BiBler's actual availability, API contract, documentation, and approved legacy fixtures before implementing; do not invent parity from this description alone (item 16 in the same register).

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [29.01 — Confirm BiBler access, documentation, fixtures, and legacy contract](sub-issues/29-01.md)
- [29.02 — The worker reads a Queued batch's temporary file and sends it to the internal service with timeouts,…](sub-issues/29-02.md)
- [29.03 — Validate responses against a fixed normalized-reference contract](sub-issues/29-03.md)
- [29.04 — Persist ordered candidates and safe warnings/errors on the existing batch without creating Papers yet](sub-issues/29-04.md)
- [29.05 — Create a preview and Source/Strategy selection before subsequent review when configured](sub-issues/29-05.md)
- [29.06 — Integrate user-approved asynchronous commit through the shared workflow](sub-issues/29-06.md)
- [29.07 — Do not persist raw responses/parser logs or intermediate debugging artifacts](sub-issues/29-07.md)
