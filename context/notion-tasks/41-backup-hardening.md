# 41 — Implement backup/restore and hardening

## Source and usage

- Source: [Implement backup/restore and hardening](https://app.notion.com/p/Implement-backup-restore-and-hardening-3b449936029380848969ecf4a33f30f9).
- Notion ID: `3b449936029380848969ecf4a33f30f9`.
- Source domain: Administration & Configuration.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Implement controlled backup and restore for individual project PostgreSQL databases, explicit confirmation for high-impact actions, optional localized completion/failure notifications, and documented data-retention/anonymization policies. Preserve the legacy ability to produce a database backup, but replace its hard-coded local path and lack of a controlled restore flow with verified, encrypted, project-isolated operations.
- **Functional Domain:** Administration & Configuration
- **Behavior:**
  - Legacy `Home.php::backup_db` implements a MySQL database dump: builds SQL from the current connection, writes an SQL file to a hard-coded local XAMPP path, optionally supports gzip, and creates a legacy backup log entry.
  - The inspected legacy controller includes backup generation but no corresponding controlled restore workflow — a ReLiS2.0 restore flow is therefore a new hardening capability, not legacy parity.
  - Legacy project publication/reopening, log clearing, operation cancellation, and project deletion-style actions are high-impact flows with uneven confirmation behaviour. `Manager.php::clear_logs_validation` is one example of an explicit warning/continue/cancel screen.
  - `Apiquery.php` accepts a `sql` query parameter and runs a wrapped query — this is a critical security risk and must not be migrated as a general SQL console/API.
  - No clear legacy retention/anonymization policy was found; this is new ReLiS2.0 compliance work.
- **Users and permissions:**
  - Application Administrator alone can request, download, restore, verify, retain, or destroy project backups; all actions require step-up confirmation and audit events.
  - Project Manager and Standard User have no backup/restore or retention-policy access.
  - Project roles: Project Admin can request a backup for their own project only if the product policy delegates that capability; they cannot restore into production or change retention policy. Reviewer, Validator, and Guest have no access to backup, restore, retention, or security-hardening actions.
- **UI behavior:**
  - Provide an Application Administrator Backups screen listing project, backup type, data snapshot time, checksum, encryption/key status, storage region/status, requester, expiry, verification result, and permitted download/restore actions.
  - Provide a restore wizard requiring explicit project selection, source backup selection, target environment, pre-restore validation, impact summary, typed confirmation, and post-restore verification result. Production restore must not silently overwrite a live project.
  - Provide a reusable high-impact confirmation dialog for archive, delete, reset, publish/reopen, backup restore, retention purge, and any future exceptional diagnostic action — showing impact, irreversible effects, a required typed phrase, and an optional reason.
  - Provide an administrator-only retention policy screen showing categories, retention duration, legal/research-data warning, scheduled actions, exceptions/holds, and anonymization preview.
  - Do not provide a generic SQL console. Offer predefined, read-only diagnostics/health checks only if operational support requires them.
- **API behavior:** Implement stable use cases: `requestProjectBackup`; `getBackupStatus`; `listProjectBackups`; `downloadBackupArtifact`; `validateRestorePlan`; `requestProjectRestore`; `verifyRestoredProject`; `configureRetentionPolicy`; `previewRetentionAction`; `executeRetentionAction`; `requestSensitiveActionConfirmation`; `sendAdministrativeNotification`. Restore use cases require a target environment and a one-time confirmation token bound to the actor, backup, target, and expiry. There is no `runSql` use case in ReLiS2.0.
- **Data model:**
  - Data lives in Both databases and controlled backup storage. The Control DB holds backup catalogues, encrypted artifact metadata, checksum/verification state, restore plans/runs, retention policies, confirmation tokens, notification records, and global audit events.
  - Each Project DB remains the data source for its own backup and may record project-scoped backup/restore audit references; backups must never combine two projects by default.
  - Backup artifacts live in encrypted controlled storage with opaque object keys, key/version metadata, checksum, expiry, and access state.
  - Store retention rules and legal-hold/anonymization decisions as versioned policy records, never as a destructive unlogged cron configuration.
- **Acceptance criteria:**
  - [ ] An Application Administrator can create an asynchronous, encrypted, checksum-verified backup of one selected project database without including another project's data.
  - [ ] Authorized backup downloads use opaque artifact IDs and permission checks; raw storage paths and predictable legacy filenames cannot grant access.
  - [ ] Restore is available only through a validated plan, explicit typed confirmation, controlled target environment, compatibility check, audit record, and post-restore verification.
  - [ ] A production restore creates a fresh safety backup and cannot silently overwrite a project.
  - [ ] High-impact actions require a common confirmation flow with impact disclosure, authorization re-check, reason where required, and audit logging.
  - [ ] Retention/anonymization policies are versioned, documented, scheduled through jobs, and respect holds/approved exceptions.
  - [ ] Generic SQL query execution from the legacy `Apiquery.php` is not exposed in ReLiS2.0; any diagnostics are allow-listed, read-only, and audited.
  - [ ] Optional completion/failure emails are localized, configurable, and contain no secrets or artifact access links.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Implement single-project backup/restore with encryption, verification, confirmation, and retention policy.

## Proposed prerequisites

- [06 — Complete job queue and progress platform](06-job-platform.md)
- [07 — Implement upload/storage service](07-storage.md)
- [16 — Prepare and Provision an Isolated Project Database Template](16-project-template.md)
- [17 — Implement project database resolver](17-project-resolver.md)
- [22 — Implement audit and undo UI](22-audit-undo.md)
- [23 — Implement system settings and custom labels](23-system-settings.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Define the Control DB catalog, encrypted-object metadata/checksum/key version/expiry, restore plans/runs, retention, holds, and audit.
2. Implement `requestProjectBackup`, `getBackupStatus`, `listProjectBackups`, and `downloadBackupArtifact` using jobs and controlled storage without mixing projects.
3. Implement `validateRestorePlan`, `requestProjectRestore`, and `verifyRestoredProject`: target environment, compatibility, single-use actor/backup/target/expiry-bound token, and typed confirmation.
4. In production, create a safety backup and perform post-restore verification; never silently overwrite a live project.
5. Create Backups, a restore wizard, and reusable high-impact confirmation; only Application Administrators manage them unless backup requests are explicitly delegated to the project's administrator.
6. Implement `configureRetentionPolicy`, `previewRetentionAction`, and `executeRetentionAction` with versioned policies, jobs, and respect for holds/exceptions.
7. Send optional localized notifications without secrets or artifact access links; provide no `runSql` API or generic SQL console.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the extensive Data model/API behavior fields — this is one of the highest-risk tickets in the set (destructive/irreversible actions), so verify each point carefully before considering it done.

- [ ] `requestProjectBackup` runs asynchronously (via [06](06-job-platform.md)) and produces an encrypted, checksum-verified backup of **exactly one** selected project's database — a backup artifact never combines two projects by default.
- [ ] `downloadBackupArtifact` is gated by opaque artifact IDs plus a permission check — no raw storage path, provider identifier, or predictable legacy-style filename can grant access on its own.
- [ ] `requestProjectRestore` only proceeds through: a `validateRestorePlan` compatibility check, an explicit target environment selection, a one-time confirmation token bound to (actor, backup, target, expiry), and a typed confirmation phrase in the UI — skipping any of these steps is impossible via the API, not just hidden in the UI.
- [ ] A restore targeting production automatically creates a fresh safety backup of the current state **before** applying the restore, and can never silently overwrite a live project without that safety net.
- [ ] `verifyRestoredProject` runs post-restore and its result is recorded and surfaced — a restore is not considered complete until this verification passes.
- [ ] Every high-impact action in this ticket (backup restore, retention purge — and per the UI behavior, also archive/delete/reset/publish/reopen from other tickets) is routed through the **same reusable confirmation dialog**: impact summary, irreversible-effects statement, a required typed phrase, and an optional reason, plus a server-side authorization re-check (not just trusting the client already showed the dialog).
- [ ] Only Application Administrators can request/download/restore/verify/retain/destroy backups by default; a Project Admin can request a backup for their own project **only if** the product explicitly delegates that capability — verify which policy was actually implemented (see Gaps) and that a Project Admin can never restore into production or change retention policy regardless.
- [ ] Retention/anonymization policies are stored as versioned policy records (never as an unlogged destructive cron job), are previewable (`previewRetentionAction`) before execution (`executeRetentionAction`), run through the job platform, and respect any recorded legal hold or approved exception — a Paper/project under an active hold is never purged.
- [ ] There is no `runSql` use case or generic SQL console anywhere in ReLiS2.0 — the legacy `Apiquery.php` behavior is not migrated in any form; any diagnostics endpoint is allow-listed, read-only, and itself audited.
- [ ] Optional completion/failure notification emails are localized and configurable, and never contain a secret, credential, or a direct artifact-access link (a link that would let the recipient download the backup without going through the normal authorization check).
- [ ] Every sensitive action here (backup request, download, restore request, restore confirmation, retention policy change, retention execution) writes a Control DB audit event, integrating with [22 — Audit and undo UI](22-audit-undo.md).
- [ ] Automated tests cover, at minimum against a disposable/test environment (never production): a full backup→restore→verify cycle for one project; a rejected restore attempt missing the confirmation token; a rejected download using a guessed/predictable identifier; and a retention preview correctly excluding a held item.
- [ ] Report which commands were run and their actual results — for anything involving a real destructive action, state clearly whether it was exercised only against disposable test infrastructure, per [testing-policy.md](../testing-policy.md).

## Gaps and pending decisions

Concrete retention/anonymization policies (exact durations, legal basis) and the extent of backup-request delegation to Project Admins require product decisions; this guide does not invent legal retention periods or decide the delegation policy — request an explicit decision before finalizing either, and record whichever choice is made.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [41.01 — Define the Control DB catalog, encrypted-object metadata/checksum/key version/expiry, restore plans/runs,…](sub-issues/41-01.md)
- [41.02 — Implement requestProjectBackup, getBackupStatus, listProjectBackups, and downloadBackupArtifact using jobs…](sub-issues/41-02.md)
- [41.03 — Implement validateRestorePlan, requestProjectRestore, and verifyRestoredProject: target environment,…](sub-issues/41-03.md)
- [41.04 — In production, create a safety backup and perform post-restore verification](sub-issues/41-04.md)
- [41.05 — Create Backups, a restore wizard, and reusable high-impact confirmation](sub-issues/41-05.md)
- [41.06 — Implement configureRetentionPolicy, previewRetentionAction, and executeRetentionAction with versioned…](sub-issues/41-06.md)
- [41.07 — Send optional localized notifications without secrets or artifact access links](sub-issues/41-07.md)
