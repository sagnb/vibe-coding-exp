# 07 — Implement upload/storage service

## Source and usage

- Source: [Implement upload/storage service](https://app.notion.com/p/Implement-upload-storage-service-3b44993602938082a1b9c5d59ce6abc2).
- Notion ID: `3b44993602938082a1b9c5d59ce6abc2`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** No storage source implementation was detected in `relis-next`. Create one project-aware storage abstraction for uploads and generated artifacts, with validation, quarantine, access control, lifecycle metadata, and secure downloads. Import/parsing and reporting/export features use this service rather than local filesystem paths.
- **Functional Domain:** Infrastructure
- **Behavior:**
  - Legacy `Paper.php` reads uploaded BibTeX/EndNote files from PHP temporary uploads; profile and project assets use application filesystem paths.
  - `Reporting.php` writes generated CSV/BibTeX/R/Python files under `cside/export_*` and downloads by filename.
- **Users and permissions:**
  - All authenticated users can upload only where the owning feature authorizes it.
  - Project Admins control project-wide export artifacts; Application Administrators have audited support access.
  - Reviewers/Validators see only artifacts their assigned feature/workflow permits.
- **UI behavior:** Provide reusable upload states: choose, validate, uploading, quarantined, accepted, rejected, expired, and remove. Generated artifact cards show status, expiry, and permission-checked download, without raw object paths.
- **API behavior:** Implement `createUploadIntent`, `completeUpload`, `scanAndValidateUpload`, `getArtifact`, `downloadArtifact`, `revokeArtifact`, and `deleteExpiredArtifact`. Feature modules supply owner scope/type and authorization policy; the storage service owns validation and object lifecycle.
- **Data model:** Data lives in Both databases and object storage. Control DB holds shared storage/job lifecycle references; Project DB holds project artifact ownership/feature references; object storage holds opaque objects. Metadata includes checksum, MIME type, byte size, scan status, retention, uploader, and audit references.
- **Acceptance criteria:**
  - [ ] Uploads and generated exports use opaque storage identifiers, not client-visible filesystem paths.
  - [ ] Type, size, checksum, and malware/virus policy are validated before an uploaded artifact becomes available.
  - [ ] Artifact ownership and download authorization enforce user/project boundaries.
  - [ ] Export artifacts support expiry/retention and job-backed status.
  - [ ] Import and reporting modules can use the service without implementing their own storage paths.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Create a storage abstraction with validation, quarantine, authorization, and upload/artifact lifecycle management.

## Proposed prerequisites

- [03 — Create control Prisma schema](03-control-prisma.md)
- [04 — Implement reusable project Prisma schema](04-project-prisma.md)
- [05 — Implement API app shell](05-api-shell.md)
- [06 — Complete job queue and progress platform](06-job-platform.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Define ownership by user/project and policies supplied by each feature.
2. Implement `createUploadIntent`, `completeUpload`, `scanAndValidateUpload`, `getArtifact`, `downloadArtifact`, `revokeArtifact`, and `deleteExpiredArtifact`.
3. Validate type, size, checksum, and malware policy before making an object available.
4. Persist lifecycle references in the Control DB and feature ownership in the Project DB as appropriate; use opaque object-storage keys.
5. Create reusable upload states and artifact cards with expiry and authorized downloads.
6. Integrate isolation, revocation, and safe failure handling.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/Data model fields.

- [ ] No filesystem path, provider identifier, or credential is ever returned by the API — every upload/artifact is referenced by an opaque storage identifier end-to-end (UI, API responses, download URLs).
- [ ] `createUploadIntent` → `completeUpload` → `scanAndValidateUpload` runs type, size, checksum, and malware/virus checks before an artifact is marked available; a rejected or quarantined artifact is never downloadable.
- [ ] `downloadArtifact`/`getArtifact`/`revokeArtifact` enforce the owning feature's authorization policy (user/project boundaries) — a user cannot fetch another user's or another project's artifact by guessing or reusing an identifier.
- [ ] Export-type artifacts support expiry/retention and reflect job-backed status (queued/processing/ready/failed) sourced from [06 — job platform](06-job-platform.md).
- [ ] `deleteExpiredArtifact` (or an equivalent scheduled/triggered cleanup) actually removes expired objects and their metadata rather than leaving orphaned references.
- [ ] Import and reporting features integrate against this service's public interface only — no feature implements its own ad hoc filesystem or bucket path.
- [ ] The reusable upload-state UI (choose/validate/uploading/quarantined/accepted/rejected/expired/remove) is used consistently wherever uploads occur, rather than each feature building its own upload widget.
- [ ] Automated tests cover: a valid upload reaching "accepted"; an invalid type/size/checksum being rejected; authorization denial for a cross-user/cross-project download attempt; and expiry-triggered cleanup.
- [ ] Report which commands were run and their actual results, including any scanning/malware-check step that could not be exercised in this environment.

## Gaps and pending decisions

This task anticipates persisted export artifacts, but later specific tasks (39, 40) require streaming without persistence. Do not impose storage on those exports. Imports require storage-level expiry without an application cleanup job.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [07.01 — Define ownership by user/project and policies supplied by each feature](sub-issues/07-01.md)
- [07.02 — Implement createUploadIntent, completeUpload, scanAndValidateUpload, getArtifact, downloadArtifact,…](sub-issues/07-02.md)
- [07.03 — Validate type, size, checksum, and malware policy before making an object available](sub-issues/07-03.md)
- [07.04 — Persist lifecycle references in the Control DB and feature ownership in the Project DB as appropriate](sub-issues/07-04.md)
- [07.05 — Create reusable upload states and artifact cards with expiry and authorized downloads](sub-issues/07-05.md)
- [07.06 — Integrate isolation, revocation, and safe failure handling](sub-issues/07-06.md)
