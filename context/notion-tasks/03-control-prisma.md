# 03 — Create control Prisma schema

## Source and usage

- Source: [Create control Prisma schema](https://app.notion.com/p/Create-control-Prisma-schema-3b4499360293800a83d1cb4880f40e10).
- Notion ID: `3b4499360293800a83d1cb4880f40e10`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** `relis-next` already contains the `@relis/database` package and Prisma dependencies, but its source is currently an empty export and no Prisma schema or migration exists. Implement the versioned Control DB schema in that existing package; do not recreate the package scaffold.
- **Functional Domain:** Infrastructure
- **Behavior:**
  - Legacy `deployment/db/initial_db.sql` seeds global users, projects, configuration, and related tables.
  - Legacy selects a project database from session/metadata and keeps global user/project records separate from project runtime tables.
- **Users and permissions:** Empty
- **UI behavior:** No direct schema UI. User, Project, Administration, and audit screens consume the schema through their owned modules.
- **API behavior:** Expose repository/use-case interfaces; do not expose generic CRUD or raw database access.
- **Data model:** Empty
- **Acceptance criteria:** Empty
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Implement the versioned Control DB schema in the existing database package, exposing repository and use-case interfaces.

## Proposed prerequisites

- [01 — Implement runtime config validation](01-runtime-config.md)
- [02 — Add Docker Compose local stack](02-docker-compose.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Locate `@relis/database` in the target repository; Notion states it already exists, but this must be verified.
2. Consolidate identity, session, project, membership, invitation, draft, job, and audit contracts from the consuming tasks.
3. Resolve the PROJECT_OWNER/PROJECT_ADMIN discrepancy before finalizing identity and membership enums and constraints.
4. Create the PostgreSQL schema, static migrations, and Control DB Prisma client.
5. Expose typed repositories; do not publish generic CRUD or raw database access.

## Definition of Done — completion checklist

Notion's Acceptance criteria property is empty; the checklist below combines that fact with concrete points grounded in the Description/Behavior/API behavior fields above.

- [ ] The schema is implemented inside the existing `@relis/database` package (its current empty-export source), not a newly scaffolded package.
- [ ] Prisma migrations apply cleanly to an empty PostgreSQL database and the client generates successfully.
- [ ] Global/control-plane entities (users, projects, memberships, sessions, invitations, drafts, jobs, audit events) live only in the Control DB — no project-runtime data is mixed in.
- [ ] The package exposes typed repository/use-case interfaces to its consumers; it does not expose a generic CRUD client or raw `PrismaClient` for ad hoc queries from other packages/apps.
- [ ] Uniqueness and integrity constraints required by consuming tasks (e.g. unique username/email, unique project short name) are present and verified with a test.
- [ ] The role/identity naming conflict between `PROJECT_ADMIN` (used in guide 15) and the protected `PROJECT_OWNER` (used in guide 18) is resolved or explicitly deferred with a recorded decision before finalizing enums — see [pending decisions](PENDING-DECISIONS.md).
- [ ] Automated tests cover migration application and at least the core repository operations consumed by later tasks (e.g. user/session/project lookups).
- [ ] Report which commands were run (migration, typecheck, tests) and their actual results.

## Gaps and pending decisions

The source does not specify entities in Data model or provide acceptance criteria. The entity list and checks synthesize consuming tasks; they are not an approved schema.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [03.01 — Locate @relis/database in the target repository](sub-issues/03-01.md)
- [03.02 — Consolidate identity, session, project, membership, invitation, draft, job, and audit contracts from the…](sub-issues/03-02.md)
- [03.03 — Resolve the PROJECT_OWNER/PROJECT_ADMIN discrepancy before finalizing identity and membership enums and…](sub-issues/03-03.md)
- [03.04 — Create the PostgreSQL schema, static migrations, and Control DB Prisma client](sub-issues/03-04.md)
- [03.05 — Expose typed repositories](sub-issues/03-05.md)
