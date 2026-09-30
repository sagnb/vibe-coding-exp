# 22 — Implement audit and undo UI

## Source and usage

- Source: [Implement audit and undo UI](https://app.notion.com/p/Implement-audit-and-undo-UI-3b449936029380e38ea4d724f7f70e22).
- Notion ID: `3b449936029380e38ea4d724f7f70e22`.
- Source domain: Administration & Configuration.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Implement a unified, append-only audit trail across the control database and each isolated project database, together with a safe undo centre for explicitly supported bulk operations. ReLiS2.0 must preserve legacy operation-batch reversal where it is safe, while replacing direct state reactivation and mutable/deletable logs with audited, permission-checked compensating actions.
- **Functional Domain:** Administration & Configuration
- **Behavior:**
  - Legacy `set_log` in `bm_helper.php` records a log type, event text, timestamp, user, IP address, optional user agent, and optional post/target identifier. `logs_config.php` exposes a list ordered by most recent log and searchable by user.
  - Legacy project operations are stored with an operation code, type, description, state, actor, and time in the project `operations` table. The operations list exposes an Undo link.
  - `Manager.php::cancel_operation` supports cancellation/re-activation for paper import; screening and screening-validation assignments; Quality Assessment assignments and validation assignments; and classification/data-extraction assignments and validation assignments. It changes matching records' active flags using the operation code, then marks the operation `Cancelled` or `Active`.
  - The legacy UI includes a confirmation screen before clearing logs, but the clear action deactivates all active logs. It is not an immutable audit design.
  - Legacy logs exist globally and operations are project-specific. Neither system provides a uniform, structured audit event across every sensitive feature.
- **Users and permissions:**
  - Application Administrator can search audit events and operation history across all projects, initiate eligible global compensating actions, and export audit results. Every such access and action is audited.
  - Project Manager has no audit/undo permission by default; limited rights must be explicitly granted in the project permission matrix.
  - Standard User has no audit centre access.
  - Project roles: Project Admin can view audit events and undo eligible operations in their own project only. Reviewer and Validator can see neither project-wide audit events nor undo controls by default. Guest has no audit or undo access.
- **UI behavior:**
  - Provide an Audit log view with filters for scope (control/project), actor, action, target type/ID, project, time range, outcome, and correlation/operation batch ID.
  - Provide event detail showing the actor, timestamp, source context, safe before/after summary, reason, and links to the affected project resource where access permits.
  - Provide an Undo centre that lists only actions with a registered compensating handler. Each row shows why it is or is not still undoable, affected item count, dependencies detected, expiry if any, and a confirmation dialog requiring a reason.
  - Provide Project Admins only their own project's view; Application Administrators can switch scope with an unmistakable project-context indicator.
  - Never provide a UI action to erase audit history. Retention or anonymization is handled through the dedicated hardening policy ([41](41-backup-hardening.md)).
- **API behavior:** Implement stable use cases: `recordAuditEvent`; `searchAuditEvents`; `getAuditEventDetail`; `listUndoableOperations`; `previewCompensatingAction`; `executeCompensatingAction`; `exportAuditEvents`. Each sensitive domain write emits a structured event using a common envelope: actor, scope, project ID when applicable, action, target, outcome, correlation ID, timestamp, and redacted metadata. `executeCompensatingAction` must validate the original action, authorization, current state, and dependency rules in one transaction.
- **Data model:**
  - Data lives in Both databases. The Control DB holds application-level audit events, global configuration events, and cross-project operation correlation metadata. Each Project DB holds project-resource events and `OperationBatch`/`CompensatingAction` records linked to the original import, assignment, archive, or configuration change.
  - `AuditEvent` is append-only. Store a schema version and redacted structured metadata rather than unbounded raw request payloads or credentials.
  - `OperationBatch` records the original action, initiator, affected-resource snapshot/count, state, cancellation/compensation references, expiry, and reason.
- **Acceptance criteria:**
  - [ ] Every sensitive control-plane and project-runtime mutation creates a structured, immutable audit event with actor, scope, target, action, outcome, timestamp, and correlation ID.
  - [ ] An Application Administrator can filter audit events across projects; a Project Admin can filter only events within their own project.
  - [ ] The undo centre lists only registered, eligible compensating actions and explains why ineligible actions cannot be undone.
  - [ ] A supported undo validates permissions and dependencies, records a reason, runs transactionally or through an idempotent job, and creates linked audit events.
  - [ ] Imported papers and unstarted assignment batches can be safely compensated only when no protected downstream work would be lost or corrupted.
  - [ ] No UI or API endpoint can delete audit history through the legacy "clear logs" behaviour.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Provide append-only auditing and safe compensation for registered operations.

## Proposed prerequisites

- [03 — Create control Prisma schema](03-control-prisma.md)
- [04 — Implement reusable project Prisma schema](04-project-prisma.md)
- [06 — Complete job queue and progress platform](06-job-platform.md)
- [15 — Implement Project Context, Roles, and Membership](15-membership.md)
- [20 — Implement Project Workspace Navigation and Home Dashboard](20-project-workspace.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Define a shared envelope: actor, scope, project, action, target, outcome, correlation ID, timestamp, schema version, and redacted metadata.
2. Integrate `recordAuditEvent` into sensitive Control/Project DB mutations; do not store credentials or raw requests.
3. Implement `searchAuditEvents`, `getAuditEventDetail`, and `exportAuditEvents` with filters and audited global or project-restricted access.
4. Create `OperationBatch`/`CompensatingAction` and explicitly registered handlers; `listUndoableOperations` explains eligibility and dependencies.
5. Implement `previewCompensatingAction`/`executeCompensatingAction` with a reason, revalidation, and a transaction or idempotent job.
6. Create Audit log and Undo centre; imports and unstarted batches can be compensated only when protected downstream work will not be destroyed.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/Data model fields.

- [ ] `recordAuditEvent` is called from every sensitive control-plane and project-runtime mutation (this is a cross-cutting requirement — verify it is wired into the mutations delivered by other already-implemented tasks, not only new ones written after this ticket) and produces an immutable event with actor, scope, project ID (when applicable), action, target, outcome, timestamp, correlation ID, and a schema version.
- [ ] `AuditEvent` records are append-only at the data-access layer — there is no update or hard-delete code path for them anywhere in the codebase.
- [ ] Event metadata is redacted/structured, never an unbounded raw request payload or credential — verify with a test that a mutation carrying a secret-shaped field never surfaces it in the recorded event.
- [ ] `searchAuditEvents`/`exportAuditEvents` scope results: an Application Administrator can filter/search across all projects; a Project Admin can filter/search only within their own project; a Project Manager has no access unless the project permission matrix explicitly grants a limited right; Reviewer/Validator/Guest/Standard User have none.
- [ ] The Undo centre lists only operations with a registered `CompensatingAction` handler; every listed (and every excluded) operation states clearly why it is or isn't still undoable (dependency detected, expiry passed, no handler registered, etc.).
- [ ] `previewCompensatingAction` and `executeCompensatingAction` both revalidate the original action, the caller's authorization, current state, and dependency rules — a preview that says "safe to undo" must still be re-checked at execution time in case state changed in between.
- [ ] `executeCompensatingAction` requires an explicit reason, runs in one transaction or through an idempotent job (per [06](06-job-platform.md)), and creates a linked audit event recording the compensation itself.
- [ ] Compensating an import or an unstarted assignment batch is rejected when doing so would lose or corrupt protected downstream work (e.g., completed screening decisions, QA assessments) — this must be an actual dependency check, not just a UI warning.
- [ ] No UI control or API endpoint can deactivate/delete audit history the way the legacy "clear logs" feature did — retention/anonymization is exclusively the domain of [41 — backup/restore and hardening](41-backup-hardening.md).
- [ ] Automated tests cover: an audit event recorded for a representative sensitive mutation; a Project Admin's search correctly excluding another project's events; a successful compensating action with its own audit trail; and a rejected compensation attempt due to protected downstream work.
- [ ] Report which commands were run and their actual results, including which already-implemented mutations (from earlier tasks) were retrofitted with `recordAuditEvent` calls versus which remain pending.

## Gaps and pending decisions

The audit foundation is required from the earliest mutations; this guide's position (22) refers to the complete UI and compensation functionality, not to when logging itself must start — do not defer audit logging for earlier features, per the [notion-tasks README](README.md)'s "work that must be split into stages" note. Project Admin and Owner require an approved mapping (see [pending decisions](PENDING-DECISIONS.md) item 2); Managers do not implicitly receive audit/undo access.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [22.01 — Define a shared envelope: actor, scope, project, action, target, outcome, correlation ID, timestamp,…](sub-issues/22-01.md)
- [22.02 — Integrate recordAuditEvent into sensitive Control/Project DB mutations](sub-issues/22-02.md)
- [22.03 — Implement searchAuditEvents, getAuditEventDetail, and exportAuditEvents with filters and audited global or…](sub-issues/22-03.md)
- [22.04 — Create OperationBatch/CompensatingAction and explicitly registered handlers](sub-issues/22-04.md)
- [22.05 — Implement previewCompensatingAction/executeCompensatingAction with a reason, revalidation, and a…](sub-issues/22-05.md)
- [22.06 — Create Audit log and Undo centre](sub-issues/22-06.md)
