# 42 — Complete CI/CD and production deployment specification

## Source and usage

- Source: [Complete CI/CD and production deployment specification](https://app.notion.com/p/Complete-CI-CD-and-production-deployment-specification-3b4499360293806a917ff0c8526bd3d6).
- Notion ID: `3b4499360293806a917ff0c8526bd3d6`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** `relis-next` has no Docker Compose or CI workflow detected yet. Deliver repeatable production deployment documentation and CI/CD quality gates for its existing web app, API, worker, PostgreSQL Control/Project DBs, object storage, reverse proxy, backups, migrations, and rollback/recovery procedures.
- **Functional Domain:** Infrastructure
- **Behavior:**
  - Legacy production/development deployment is described by Docker Compose, PHP/Tomcat images, Nginx configuration, and manual database assets.
  - Legacy CI builds Docker Compose, waits a fixed time, calls a test endpoint, then uploads an HTML result.
- **Users and permissions:** Empty
- **UI behavior:** Empty
- **API behavior:** Empty
- **Data model:** Empty
- **Acceptance criteria:**
  - [ ] CI runs lint, typecheck, tests, Prisma migration checks, parity suite, and container smoke tests before merge/release.
  - [ ] Production deployment is documented and repeatable for web, API, worker, PostgreSQL, storage, reverse proxy, mail, and backups.
  - [ ] Secrets are externalized, rotated safely, and never written to logs/artifacts.
  - [ ] Release runbooks cover Control DB and Project DB migrations, job draining, health verification, backup, rollback, and incident recovery.
  - [ ] Deployment verifies project workspace schema compatibility without exposing project data.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Document repeatable deployment and configure quality and recovery gates.

## Proposed prerequisites

- [02 — Add Docker Compose local stack](02-docker-compose.md)
- [03 — Create control Prisma schema](03-control-prisma.md)
- [04 — Implement reusable project Prisma schema](04-project-prisma.md)
- [06 — Complete job queue and progress platform](06-job-platform.md)
- [16 — Prepare and Provision an Isolated Project Database Template](16-project-template.md)
- [22 — Implement audit and undo UI](22-audit-undo.md)
- [41 — Implement backup/restore and hardening](41-backup-hardening.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Create baseline CI early using actual scripts: lint, typecheck, tests, Prisma migration checks, parity, and container smoke tests.
2. Document web/API/worker, Control/Project PostgreSQL, object storage, proxy, mail, and backups.
3. Externalize secrets and document rotation without leakage into logs/artifacts.
4. Define a release runbook with Control/Project migrations, template publication, compatibility, job draining, and health/readiness.
5. Document backup, rollback, incident recovery, and isolation verification without exposing data.
6. Rehearse the runbook in a test environment and record results; actual deployment is separate from writing this guide.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points. This card's Behavior/UI/API/Data model properties are all empty, so the checklist relies on Acceptance criteria and the step-by-step sequence above.

- [ ] CI actually runs, and blocks merge/release on failure of: lint, typecheck, the test suite, a Prisma migration check (both Control and Project schemas), the parity suite (where one exists), and a container smoke test — wired to real scripts, not placeholder steps.
- [ ] Production deployment is documented as a repeatable procedure covering: web, API, worker, PostgreSQL (Control + Project template), object storage, reverse proxy, mail, and backups — someone unfamiliar with the system could follow it to stand up an environment.
- [ ] Secrets used anywhere in CI/CD or deployment are externalized (never committed) and have a documented rotation procedure; verify no secret appears in a log line or a build artifact.
- [ ] A release runbook exists covering: Control DB migrations; Project DB / `relis_project_template` migrations and republishing (per [16](16-project-template.md), a release-process concern, never part of end-user project creation); job draining before deploy (per [06](06-job-platform.md)); post-deploy health/readiness verification (per [05](05-api-shell.md)); backup before a risky release (per [41](41-backup-hardening.md)); rollback steps; and incident recovery steps.
- [ ] Deployment includes a step that verifies project workspace schema compatibility (i.e., that already-provisioned project databases remain compatible with the new application version) without exposing any actual project data during that check.
- [ ] The runbook has actually been rehearsed at least once in a test/disposable environment, and the rehearsal's results (what worked, what didn't) are recorded — writing the document alone is not sufficient.
- [ ] Basic CI gates were established early (lint/typecheck/tests) rather than deferred to the end of the project, per the "work that must be split into stages" guidance in the [notion-tasks README](README.md); this ticket's position in the numbered sequence (42) represents completing the full release/recovery gates, not the earliest possible CI setup.
- [ ] Report which commands were actually run (the CI pipeline itself, or a local rehearsal of its steps) and their actual results, distinguishing "documented" from "verified by rehearsal."

## Gaps and pending decisions

Basic CI should start early (as soon as executable scripts exist); this task's position (42) represents operational completion of the full release/recovery gates, not the starting point — see the [notion-tasks README](README.md), "work that must be split into stages." Updating templates/migrating databases is a release process, never part of end-user project creation, per [pending decisions](PENDING-DECISIONS.md) item 5.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [42.01 — Create baseline CI early using actual scripts: lint, typecheck, tests, Prisma migration checks, parity,…](sub-issues/42-01.md)
- [42.02 — Document web/API/worker, Control/Project PostgreSQL, object storage, proxy, mail, and backups](sub-issues/42-02.md)
- [42.03 — Externalize secrets and document rotation without leakage into logs/artifacts](sub-issues/42-03.md)
- [42.04 — Define a release runbook with Control/Project migrations, template publication, compatibility, job…](sub-issues/42-04.md)
- [42.05 — Document backup, rollback, incident recovery, and isolation verification without exposing data](sub-issues/42-05.md)
- [42.06 — Rehearse the runbook in a test environment and record results](sub-issues/42-06.md)
