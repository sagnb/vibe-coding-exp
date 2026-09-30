# 16 — Prepare and Provision an Isolated Project Database Template

## Source and usage

- Source: [Prepare and Provision an Isolated Project Database Template](https://app.notion.com/p/Prepare-and-Provision-an-Isolated-Project-Database-Template-3bc49936029380bfa661f166af65c367).
- Notion ID: `3bc49936029380bfa661f166af65c367`.
- Source domain: Project & Protocol Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Build the Native Project Protocol Wizard.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Prepare a fixed PostgreSQL project-database template and provision one isolated Project DB by cloning that template for each new ReLiS project. The project-creation flow must never run database migrations or create database structures dynamically.
- **Functional Domain:** Project & Protocol Management
- **Behavior:** Legacy ReLiS created a dedicated MySQL database for each installed project and generated project-specific tables, views, procedures, and configuration from DSL/editor files. NeoReLiS preserves one database per project, but replaces generated structures with a prebuilt PostgreSQL template containing one fixed, reusable ReLiS schema.
- **Users and permissions:**
  - Only authenticated users authorized to create projects can start the project-creation workflow.
  - The creator receives a pending Project Manager membership in the Control DB.
  - Only backend provisioning services can access database credentials or create/clone Project DBs.
- **UI behavior:**
  - The Project Creation Wizard can display Provisioning project database, Ready, or Failed.
  - Authorized users can retry a failed creation safely.
  - The UI never displays PostgreSQL errors, database identifiers, connection strings, or credentials.
- **API behavior:**
  - Hono exposes authenticated endpoints to start and retrieve provisioning status. It creates one idempotent provisioning request per project-creation command and delegates physical database work to an application service or worker.
  - The provisioning service generates the Project DB identifier from the internal project UUID, never from the project title or short name.
  - Maintain a prebuilt PostgreSQL database named `relis_project_template` outside the end-user project-creation flow. Provision the isolated Project DB by cloning this template.
  - Verify that the cloned database is reachable and compatible with the running application's fixed Prisma data model.
  - Do not run `prisma migrate deploy`, `prisma db push`, generated SQL, schema alterations, or dynamic table creation during project creation.
  - Protocol categories, QA questions, screening phases, sources, strategies, and reporting definitions create configuration rows only. They must never create tables, columns, views, procedures, Prisma models, or migrations.
  - Prisma is used only for normal CRUD against the cloned database's existing fixed schema.
  - Resolve a Project DB connection only after the project is ready and the caller passes project-state and membership authorization checks. Never expose database identifiers, URLs, credentials, or connection details through API responses.
- **Data model:**
  - Control DB: project registry, project short name, provisioning state, idempotency key, pending Project Manager membership, template release identifier, job reference, safe failure code/message, timestamps, and audit events.
  - Project DB template: a complete fixed ReLiS schema with empty project-runtime tables and no project-specific users, papers, protocol configuration, review decisions, or credentials.
  - Cloned Project DB: the same fixed schema as the template. The Project Creation Wizard ticket later inserts the validated v1 protocol as rows in these existing tables.
- **Acceptance criteria:**
  - [ ] A maintained `relis_project_template` database exists and contains the complete fixed Project DB schema with no project data.
  - [ ] Creating a project provisions exactly one isolated PostgreSQL database by cloning the template.
  - [ ] No migration, db push, generated SQL, or dynamic schema creation runs during project creation.
  - [ ] Two projects cannot read or write each other's Project DB data.
  - [ ] Repeating the same provisioning command or retrying after failure does not create another database, membership, or project record.
  - [ ] The frontend receives only safe provisioning state and error information.
  - [ ] Prisma can connect to a cloned Project DB and use the fixed data model successfully.
  - [ ] The cloned workspace is ready for the Project Creation workflow to persist protocol version v1.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Build the Native Project Protocol Wizard
- **Jobs behavior:** The worker creates or reuses the project database clone, verifies connectivity, and records a safe result. The operation is idempotent: a retry must reuse an already-cloned Project DB for the same project; it must never clone a second database for the same project; it must not create duplicate Control DB records or memberships. The project remains in provisioning until the Project Creation workflow persists the validated protocol v1 and activates the project. A terminal failure records a non-sensitive reason and supports an explicit retry.
- **Audit:** Empty

## Objective and summarized requirements

Provision exactly one isolated PostgreSQL database per project by cloning a fixed template.

## Proposed prerequisites

- [03 — Create control Prisma schema](03-control-prisma.md)
- [04 — Implement reusable project Prisma schema](04-project-prisma.md)
- [06 — Complete job queue and progress platform](06-job-platform.md)
- [15 — Implement Project Context, Roles, and Membership](15-membership.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Prepare `relis_project_template` outside the creation flow, with a complete schema and no project data, credentials, or users.
2. Implement idempotent requests and status in the Control DB; derive the database identifier from the internal UUID.
3. In the worker, clone or reuse the database already created for the same project and verify connectivity/Prisma compatibility.
4. Do not run migrate deploy, db push, or schema changes during creation; categories and protocols create records only.
5. Record template release, job, safe failures, timestamps, and audit events; retries must not duplicate databases/projects/memberships.
6. Keep the project in provisioning until the wizard persists v1 and activates the membership/project; the API returns safe status only.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/Jobs behavior fields.

- [ ] `relis_project_template` exists as a maintained PostgreSQL database containing the complete fixed schema from [04](04-project-prisma.md), with zero project data, users, or credentials — maintained outside the end-user creation flow (a release/ops process, not something project creation itself builds).
- [ ] Project creation clones this template to produce exactly one new isolated database per project; the database identifier is derived from the internal project UUID, never from the title or short name.
- [ ] No `prisma migrate deploy`, `prisma db push`, generated SQL, or dynamic DDL runs as part of project creation — protocol categories/QA questions/screening phases/sources/strategies/report definitions insert configuration **rows** only.
- [ ] Two provisioned projects cannot read or write each other's Project DB data (verified with a cross-project access test once [17](17-project-resolver.md) exists).
- [ ] The provisioning request is idempotent: repeating the same command, or retrying after a failure, reuses the already-cloned database and does not create a second database, a duplicate Control DB project record, or a duplicate pending membership.
- [ ] The frontend only ever sees `Provisioning`, `Ready`, or `Failed` (plus a safe retry action) — no PostgreSQL error text, database identifier, connection string, or credential is ever returned to the client.
- [ ] After cloning, the worker verifies the new database is reachable and Prisma-compatible with the current fixed data model before marking it ready.
- [ ] The project stays in `Provisioning` state until [18 — the Protocol Wizard](18-protocol-wizard.md) has persisted protocol version `v1` and the workflow explicitly activates the project and the creator's membership — this task does not activate anything on its own.
- [ ] A terminal failure records a non-sensitive reason and exposes an explicit, safe retry action; retrying after a terminal failure follows the same idempotency guarantees above.
- [ ] Automated tests cover: successful clone + connectivity check; idempotent retry not duplicating a database; and a simulated clone failure surfacing a safe error with retry available.
- [ ] Report which commands were run and their actual results, and flag the circular dependency with [18](18-protocol-wizard.md) explicitly rather than silently resolving it (see Gaps below).

## Gaps and pending decisions

There is an explicit circular dependency with the wizard ([18](18-protocol-wizard.md) is `Blocked by` this task, and this task is `Blocked by` the wizard). Proposal: define a shared contract, implement the clone service, then orchestration. The source uses a pending Project Manager membership, conflicting with the wizard's protected `PROJECT_OWNER` — see [pending decisions](PENDING-DECISIONS.md).

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [16.01 — Prepare relis_project_template outside the creation flow, with a complete schema and no project data,…](sub-issues/16-01.md)
- [16.02 — Implement idempotent requests and status in the Control DB](sub-issues/16-02.md)
- [16.03 — In the worker, clone or reuse the database already created for the same project and verify…](sub-issues/16-03.md)
- [16.04 — Do not run migrate deploy, db push, or schema changes during creation](sub-issues/16-04.md)
- [16.05 — Record template release, job, safe failures, timestamps, and audit events](sub-issues/16-05.md)
- [16.06 — Keep the project in provisioning until the wizard persists v1 and activates the membership/project](sub-issues/16-06.md)
