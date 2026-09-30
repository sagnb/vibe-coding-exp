# 01 — Implement runtime config validation

## Source and usage

- Source: [Implement runtime config validation](https://app.notion.com/p/Implement-runtime-config-validation-3b4499360293802cbed7fada48d48d0a).
- Notion ID: `3b4499360293802cbed7fada48d48d0a`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file now reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist. It is still not a verbatim Notion export/transcription tool output — read the original page for any subsequent edits.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Define one typed environment configuration contract for web, API, worker, migration, and deployment processes, and fail fast before accepting traffic or writing data when required configuration is absent or invalid.
- **Functional Domain:** Infrastructure
- **Behavior:**
  - Legacy ReLiS uses PHP configuration files, deployment `.env`, database configuration, and installer checks.
  - Configuration is split among application code, Docker files, and external-editor settings.
- **Users and permissions:** Empty
- **UI behavior:** Empty
- **API behavior:** Expose health/readiness diagnostics that identify a configuration category without leaking values. All processes load the same validated contract.
- **Data model:** Empty
- **Acceptance criteria:** Empty
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Define one typed configuration contract for web, API, worker, migrations, and deployment; fail before accepting traffic or writing data when configuration is invalid.

## Proposed prerequisites

No mandatory preceding guide; confirm the repository context.

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Inventory the variables used by each process and separate public values from secrets.
2. Define the shared configuration contract and required validation; Zod is the library specified in the stack.
3. Integrate validated configuration loading into each process startup, before connections and writes.
4. Expose health/readiness diagnostics by configuration category without returning values.
5. Document variables and examples without real credentials.

## Definition of Done — completion checklist

Notion's Acceptance criteria property is empty; the checklist below combines that fact with concrete points grounded in the Description and API behavior text above, plus the repository's own testing/verification standards.

Status as of the last implementation session (see `claude-sessions/` if archived): web, API, and worker were implemented and corrected across several sessions; migration and deployment were completed in the final session, each against the currently-empty `control`/`project` schemas (no domain model exists or was added).

- [x] One shared, typed (Zod) configuration contract is used by web, API, worker, migration, and deployment tooling — no process parses its own ad hoc env access. (`packages/config`; migration via `loadDatabaseConfigForTarget`, deployment via the same api/web/worker/database loaders plus `checkPortConflicts`.)
- [x] Missing or invalid required configuration prevents the affected process from starting or accepting traffic/writes ("fail fast"), instead of running in a partially configured state. Verified for web/API/worker in earlier sessions; migration (`packages/database/src/migrate.ts`) and deployment (`tooling/scripts/deploy.mjs`) both validate before invoking Prisma or starting/building/spawning anything.
- [x] `/health` and `/readiness`-style diagnostics report validity per configuration category without ever returning the actual values. (API's `/health` + `/ready`; migration/deployment diagnostics use the same category/variable-name-only shape, no HTTP endpoint since neither is a server.)
- [x] Secrets are never logged, returned in diagnostics, or committed to the repository; `.env.example` documents variable names with placeholder, non-real values, including `CONTROL_DATABASE_URL`/`PROJECT_DATABASE_URL`.
- [x] The contract distinguishes public/non-sensitive values from secrets (`packages/config/src/public.ts` stays free of Node/server-only imports, verified by keeping IPv6/hostname validation in a separate server-only module).
- [x] Automated tests cover: valid configuration loads successfully; each required variable's absence or malformed value is rejected with a clear error; diagnostics never echo secret values. Extended in the final session to migration (`tests/integration/runtime-config/migration.test.ts`) and deployment (`.../deployment.test.ts`), using injected fakes for ordering/supervision checks and real (but database-free, deployment-free) subprocesses for entry-point behavior.
- [x] Report which commands were actually run (typecheck/tests), their results, and anything that could not be verified — see each session's final report; the last one lists exact commands/results and remaining limitations.

**Remaining limitations (not resolved by this card, reported rather than hidden):**
- Neither `packages/database/prisma/control/schema.prisma` nor `.../project/schema.prisma` defines a model — no control-plane or project-data domain has been authorized. Migration is a real, runnable command for these empty schemas, not a business-schema implementation.
- Migration/deployment were verified with dependency-injected fakes and real subprocesses against isolated fixtures and deliberately-invalid config; a real Postgres connection was deliberately never used (one test does exercise real Prisma CLI tooling against a guaranteed-unreachable port to prove failure propagation, still without touching a database).
- `apps/web`'s default build (Turbopack) fails in the sandbox this work was done in, due to a Windows Application Control Policy unrelated to this task; `--webpack` was used to verify product behavior, and this is tracked as a skipped/blocked check, not a pass.
- A pre-existing, unrelated `apps/web` lint failure (`react-hooks/set-state-in-effect`) remains unfixed (out of scope).
- Deployment does not build/push container images, gate on health checks, or perform rolling/zero-downtime restarts; Docker Compose (the prescribed local/dev/test runtime per `context/stack.yml`) was not scaffolded — that remains separate, larger, unauthorized-here work (see `02-docker-compose.md`).

## Gaps and pending decisions

The Notion page does not provide a data model, users/permissions, UI behavior, or acceptance criteria; the checklist above is a technical proposal grounded only in the Description and API behavior fields, which are the only non-empty requirement fields on the source card.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [01.01 — Inventory the variables used by each process and separate public values from secrets](sub-issues/01-01.md)
- [01.02 — Define the shared configuration contract and required validation](sub-issues/01-02.md)
- [01.03 — Integrate validated configuration loading into each process startup, before connections and writes](sub-issues/01-03.md)
- [01.04 — Expose health/readiness diagnostics by configuration category without returning values](sub-issues/01-04.md)
- [01.05 — Document variables and examples without real credentials](sub-issues/01-05.md)
