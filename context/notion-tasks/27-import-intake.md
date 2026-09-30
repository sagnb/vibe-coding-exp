# 27 — Implement Secure Import Storage and Project Job Progress

## Source and usage

- Source: [Implement Secure Import Storage and Project Job Progress](https://app.notion.com/p/Implement-Secure-Import-Storage-and-Project-Job-Progress-3b34993602938099afb4c759dd6a3e14).
- Notion ID: `3b34993602938099afb4c759dd6a3e14`.
- Source domain: Paper Library & Import.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Create the shared upload and background-job foundation used by CSV, BibTeX, and EndNote imports. Uploaded files are validated and stored through a storage abstraction; import progress is visible to authorized users.
- **Functional Domain:** Paper Library & Import
- **Behavior:** Legacy ReLiS uploads files and performs many long operations synchronously. NeoReLiS replaces this with validated object/file storage and durable worker jobs so imports can continue safely after a browser request ends. Import file types and maximum size follow global policy and project protocol toggles. A disabled paper-import feature rejects uploads and job creation for that project.
- **Users and permissions:**
  - Project owner and Project Managers can upload and view jobs for their project.
  - Application Administrators can inspect failed jobs across projects.
  - Other project users may view an import job only if their project capability permits it.
- **UI behavior:**
  - An authorised user can open Import papers from the project workspace and access a page for starting and tracking project import batches. The main import page contains: 1. Start a new import; 2. Import history.
  - **1. Start a new import** — choose one supported file type (CSV; BibTeX `.bib`; EndNote Tagged export `.enw`); display a file picker and drag-and-drop area; before upload, show immediate feedback (selected file name, format, extension); on confirmation, upload the file, create one tracked `ImportBatch`, show its initial server-side status, and allow the user to safely leave the page while the upload-storage job continues in the background. This ticket validates and stores the input file only — it does not parse records, map CSV columns, select batch metadata, detect duplicates, review references, commit papers, or roll back papers.
  - **2. Import history** — lists only import batches belonging to the current project; for each: safe original file name, selected format, initiating user, creation date/time, current status, and an action to open batch details. Implemented visible statuses in this ticket: `Uploading`; `Queued`; `Failed`; `Cancelled`. Use a readable label and progress indicator.
  - **Import-batch detail page** — batch identifier; safe file name; selected format; initiator; relevant timestamps; current status; current upload/storage progress. On failure: a human-readable error summary and whether technical retry is available. Show actions only when the server declares them available: Retry for an eligible technical upload/storage failure; Cancel for an eligible batch that has not yet entered a parser or commit workflow. Refreshing/reopening retrieves the current server-side batch status — the browser must never be the source of truth for import progress or file data.
- **API behavior:**
  - Use Hono as the authenticated HTTP/API layer and delegate upload and temporary-storage work to application services and workers. Enforce the current project context and import permission before every operation. This ticket handles secure file intake and `ImportBatch` tracking only.
  - Provide operations to: create an `ImportBatch` from an uploaded file; list Import Batches for the current project; retrieve one `ImportBatch` and its current upload/storage progress; retry an eligible technical upload/storage failure; cancel an eligible batch before it enters a parser or commit workflow.
  - Validate the uploaded file before accepting it: supported import type (CSV; BibTeX; EndNote export compatible with the legacy BiBler flow); configured file-size limit; safe file name; declared MIME type; basic detected content checks where applicable. Create `ImportBatch`es idempotently — repeating the same confirmed upload must not create duplicate batches, worker jobs, or stored-file records. Store each accepted file in private, project-scoped temporary storage and retain only an opaque storage reference in application records. Never expose storage paths, provider identifiers, credentials, unrestricted download URLs, or raw file content through the API. Process file storage and technical validation asynchronously. A `Queued` batch has been stored successfully and is ready for a later parser workflow — it is not a completed import; its temporary file must remain available.
  - This ticket implements only these `ImportBatch` statuses: `Uploading`; `Queued`; `Failed`; `Cancelled`. Return a client-safe batch read model: current status; upload/storage progress; safe error summary when applicable; retry eligibility; retry-expiry time when applicable; currently available actions.
  - **Temporary-file lifecycle** — delete the stored file immediately when the batch is cancelled, then mark it `Cancelled`. A later parser/commit workflow deletes the stored file immediately after a successful final import. For a retryable technical failure, retain the stored file only until its configured short expiry, initially 24 hours. For a non-retryable failure, delete the stored file immediately. Configure storage-level expiry when the file is created as a safety net for interrupted workers — this must not require an application cleanup job. Once the file has expired or been deleted, disable Retry and require a new upload. A retry reuses the same `ImportBatch` and temporary stored file only while that file remains available. The worker may apply a bounded automatic retry policy for transient technical failures before exposing `Failed`. Cancellation is available only before the batch enters a parser or commit workflow.
- **Data model:**
  - Control DB — durable worker jobs, status, bounded retry attempts, and technical audit history; each job references the internal `projectId` and `importBatchId`; keep internal diagnostics separate from user-facing error summaries.
  - Project DB — one `ImportBatch` per confirmed upload, storing: selected import format; safe original file name; initiating user; status and progress; creation/update/queue/failure/cancellation/completion timestamps; safe error summary; retry eligibility and file-expiry time; file-deleted timestamp when applicable. While active, store its opaque temporary storage reference; remove the reference after the file is deleted, keeping only the deletion timestamp for audit purposes. Do not store rollback state, raw file content, or temporary parsing artifacts. Later parser/import tickets may add result counts to the same `ImportBatch`; they must not retain the original file.
  - Private temporary file storage — store the original uploaded file only while needed by the active import workflow; set a short expiry when created, initially 24 hours; delete the file immediately after cancellation, successful final import, or non-retryable failure.
- **Acceptance criteria:**
  - [ ] Unauthorized users cannot upload or access import files for another project.
  - [ ] Invalid files are rejected before parsing with localized, actionable feedback.
  - [ ] A worker can resume/retry a durable job without duplicate commits.
  - [ ] Project Admins see live import state and a final count of processed, imported, duplicate, and failed records.
  - [ ] Storage paths and credentials are never returned to the frontend.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Receive CSV/BibTeX/EndNote files and create a tracked ImportBatch before parsing or committing records.

## Proposed prerequisites

- [06 — Complete job queue and progress platform](06-job-platform.md)
- [07 — Implement upload/storage service](07-storage.md)
- [15 — Implement Project Context, Roles, and Membership](15-membership.md)
- [17 — Implement project database resolver](17-project-resolver.md)
- [20 — Implement Project Workspace Navigation and Home Dashboard](20-project-workspace.md)
- [24 — Implement Project Paper List](24-paper-list.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Create Start a new import, Import history, and project-scoped details with CSV/.bib/.enw selection, picker/drop area, and safe filenames.
2. Validate extension/format, size, MIME, and basic content; create idempotent batch/job/storage records.
3. Store originals in private temporary storage using opaque references; workers perform technical validation/storage.
4. Implement only Uploading/Queued/Failed/Cancelled; Queued means ready for a parser, not a completed import.
5. Return progress, safe failure, retry eligibility/expiry, and allowed actions. Retry uses the same batch and available file.
6. Allow cancellation only before parsing/commit. Delete originals on cancellation, later final success, or non-retryable failure; remove the reference and record `deletedAt`.
7. Apply an initial 24-hour storage TTL to protect against interrupted workers, without a dedicated cleanup job. Require a new upload if the file is gone.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the extensive API behavior/Data model fields — note that some acceptance-criteria wording (duplicate/failed record counts) belongs to later parsing tickets, per Gaps below.

- [ ] Uploading is restricted to the Project Owner and Project Managers for their own project; other roles' visibility follows their project capability, and Application Administrators can inspect failed jobs across projects — verified per endpoint, not only hidden in the UI.
- [ ] A disabled paper-import feature/module rejects both the upload and `ImportBatch` creation for that project (server-side, not only a hidden button).
- [ ] File validation runs before acceptance: supported type (CSV/BibTeX/EndNote), configured size limit, safe file name, declared MIME type, and basic content checks — an invalid file never reaches `Queued`.
- [ ] Repeating the same confirmed upload is idempotent: it does not create a duplicate `ImportBatch`, worker job, or stored-file record (test this explicitly with a repeated request).
- [ ] Only these four statuses exist in this ticket's scope: `Uploading`, `Queued`, `Failed`, `Cancelled` — `Queued` explicitly means "stored and ready for a parser," never "import completed."
- [ ] No storage path, provider identifier, credential, or unrestricted download URL is ever returned by any endpoint — only the opaque `ImportBatch` read model (status, progress, safe error summary, retry eligibility/expiry, available actions).
- [ ] Cancel is available, and effective, only before the batch enters a parser/commit workflow; cancelling deletes the stored file immediately and marks the batch `Cancelled`.
- [ ] Retry is available only for an eligible technical failure and only while the temporary file has not expired/been deleted; once expired, Retry is disabled and a new upload is required — the same `ImportBatch`/file are reused for an eligible retry, never a new batch.
- [ ] The stored file's storage-level expiry (initially 24 hours) protects against interrupted workers without needing a dedicated application cleanup job; the file is deleted immediately on cancellation, successful final import (from a later ticket), or non-retryable failure, and `deletedAt` is recorded with the storage reference cleared.
- [ ] Refreshing or reopening the import-batch detail page always re-fetches server-side status — the browser is never authoritative for progress or file data.
- [ ] Worker job tracking (status, bounded retry attempts, technical audit history) lives in the Control DB and references the internal `projectId`/`importBatchId`, keeping internal diagnostics separate from the user-facing safe error summary stored on the Project DB `ImportBatch`.
- [ ] Automated tests cover: a successful upload reaching `Queued`; an invalid file rejected before `Queued`; a duplicate confirmed-upload not creating a second batch; an eligible cancel deleting the file; and a retry rejected once the file has expired.
- [ ] Report which commands were run and their actual results, and explicitly note that "final count of processed, imported, duplicate, and failed records" (from the Acceptance criteria list) is delivered by [28](28-csv-import.md)/[29](29-bibler-import.md), not by this intake-only ticket (see Gaps).

## Gaps and pending decisions

Acceptance criteria mention imported/duplicate counts and commits, but UI/API behavior explicitly exclude parsing, deduplication, review, and rollback from this ticket's scope. Keep intake separate and connect results through [28](28-csv-import.md) and [29](29-bibler-import.md); clarify the TTL policy for active batches that exceed 24 hours per [pending decisions](PENDING-DECISIONS.md) item 15.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [27.01 — Create Start a new import, Import history, and project-scoped details with CSV/](sub-issues/27-01.md)
- [27.02 — Validate extension/format, size, MIME, and basic content](sub-issues/27-02.md)
- [27.03 — Store originals in private temporary storage using opaque references](sub-issues/27-03.md)
- [27.04 — Implement only Uploading/Queued/Failed/Cancelled](sub-issues/27-04.md)
- [27.05 — Return progress, safe failure, retry eligibility/expiry, and allowed actions](sub-issues/27-05.md)
- [27.06 — Allow cancellation only before parsing/commit](sub-issues/27-06.md)
- [27.07 — Apply an initial 24-hour storage TTL to protect against interrupted workers, without a dedicated cleanup job](sub-issues/27-07.md)
