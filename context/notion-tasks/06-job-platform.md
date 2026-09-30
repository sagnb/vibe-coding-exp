# 06 — Complete job queue and progress platform

## Source and usage

- Source: [Complete job queue and progress platform](https://app.notion.com/p/Complete-job-queue-and-progress-platform-3b449936029380bcb3fff8a281966d35).
- Notion ID: `3b449936029380bcb3fff8a281966d35`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** `relis-next` already has an `apps/worker` package and declares `pg-boss`, but `src/main.ts` currently only logs that the worker started and contains no boss client or handlers. Complete the shared job platform used by provisioning, imports, duplicate detection, exports, reporting, backups, retention, and notifications.
- **Functional Domain:** Infrastructure
- **Behavior:**
  - Legacy controllers run long import, generation, reporting, and database operations synchronously during HTTP requests.
  - There is no common persisted job state, retry policy, or progress dashboard.
- **Users and permissions:**
  - Application Administrators see all jobs.
  - Project Admins see jobs for their own projects; other project roles see only the jobs explicitly surfaced by their assigned feature.
- **UI behavior:** Provide a Jobs page and reusable job-status component with queued/running/completed/failed/cancelled state, progress, safe message, retry eligibility, actor, project scope, and links to the owning workflow.
- **API behavior:** Implement `enqueueJob`, `getJobStatus`, `listJobs`, `retryJob`, `cancelJob`, and worker handlers. Payloads carry opaque project IDs and source references, never database credentials or raw secrets.
- **Data model:** pg-boss tables live in the Control DB. Add application job metadata for actor, project scope, progress, safe error, correlation ID, result-artifact references, idempotency key, and audit links.
- **Acceptance criteria:**
  - [ ] Feature modules can enqueue idempotent, project-scoped jobs through one platform API.
  - [ ] Application Administrators and Project Admins see only the jobs their scope permits.
  - [ ] Jobs report progress, retries, terminal failures, and result references safely.
  - [ ] Retry/cancel cannot create duplicate project provisioning, imports, or exports.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Complete the shared job platform with pg-boss, progress, retry, cancellation, and idempotency.

## Proposed prerequisites

- [02 — Add Docker Compose local stack](02-docker-compose.md)
- [03 — Create control Prisma schema](03-control-prisma.md)
- [05 — Implement API app shell](05-api-shell.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Verify `apps/worker` and the `pg-boss` dependency mentioned in Notion.
2. Implement `enqueueJob`, `getJobStatus`, `listJobs`, `retryJob`, `cancelJob`, and handler registration.
3. Keep pg-boss tables in the Control DB and metadata for actor, project, progress, safe error, correlation ID, results, and idempotency key.
4. Carry opaque IDs and references in payloads; resolve credentials on the server.
5. Create the Jobs page and queued/running/completed/failed/cancelled status component; restrict visibility by role and scope.
6. Validate retry/cancel and worker failures without duplicating effects in consuming tasks.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add implementation-level detail grounded in the Data model/API behavior fields.

- [ ] `enqueueJob`, `getJobStatus`, `listJobs`, `retryJob`, and `cancelJob` exist as one shared platform API; feature modules (provisioning, imports, exports, backups, etc.) call this API rather than each rolling its own queue logic.
- [ ] Every job enqueue accepts/derives an idempotency key so repeating the same logical request does not create a duplicate job (verified with a test that enqueues twice with the same key).
- [ ] `listJobs`/`getJobStatus` scope results: Application Administrators see all jobs; Project Admins see only jobs for their own project; other roles see only what their owning feature explicitly surfaces (no cross-project leakage).
- [ ] Job payloads carry opaque project IDs and source references only — no database credentials, connection strings, or raw secrets ever appear in a pg-boss payload or in job metadata.
- [ ] The Jobs page and its reusable status component render queued/running/completed/failed/cancelled states, progress, a safe (non-technical) error message, retry eligibility, the acting user, project scope, and a link back to the owning workflow.
- [ ] `retryJob` and `cancelJob` are safe to call on an eligible job without creating duplicate project provisioning, imports, or exports (tested against at least one real consuming workflow once available, or with a mocked handler that asserts single-execution semantics).
- [ ] pg-boss's own tables remain in the Control DB; the additional application metadata (actor, project scope, progress, safe error, correlation ID, result-artifact references, idempotency key, audit links) is modeled alongside them.
- [ ] Automated tests cover: enqueue → status transition → completion; a forced failure path; an idempotent double-enqueue; and role-scoped listing.
- [ ] Report which commands were run and their actual results, and flag the stack.yml pg-boss/BullMQ decision explicitly (see Gaps below) rather than silently assuming it is settled.

## Gaps and pending decisions

stack.yml leaves pg-boss/BullMQ undecided, but Notion selects pg-boss. Record the decision before implementation. Direct CSV/BibTeX/Python/R exports have their own requirements prohibiting jobs.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [06.01 — Verify apps/worker and the pg-boss dependency mentioned in Notion](sub-issues/06-01.md)
- [06.02 — Implement enqueueJob, getJobStatus, listJobs, retryJob, cancelJob, and handler registration](sub-issues/06-02.md)
- [06.03 — Keep pg-boss tables in the Control DB and metadata for actor, project, progress, safe error, correlation…](sub-issues/06-03.md)
- [06.04 — Carry opaque IDs and references in payloads](sub-issues/06-04.md)
- [06.05 — Create the Jobs page and queued/running/completed/failed/cancelled status component](sub-issues/06-05.md)
- [06.06 — Validate retry/cancel and worker failures without duplicating effects in consuming tasks](sub-issues/06-06.md)
