# 08 — Seed installation and demo data

## Source and usage

- Source: [Seed installation and demo data](https://app.notion.com/p/Seed-installation-and-demo-data-3b44993602938006837fc7cf286a2bf5).
- Notion ID: `3b44993602938006837fc7cf286a2bf5`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Implement an idempotent first-run installer that creates the Control DB schema, the first Application Administrator, required settings, and optional non-production demo user/project data.
- **Functional Domain:** Infrastructure
- **Behavior:**
  - Legacy `initial_db.sql` seeds initial application data, including administrator/demo information.
  - `Install.php` installs and updates project structures from configuration files and can initialize demo/project configuration.
- **Users and permissions:**
  - Before initialization, only the installation operator can run the installer.
  - The installer creates the first Application Administrator; it must not leave an initialized system without one.
- **UI behavior:** Provide a protected first-run/setup screen with environment validation result, first-admin creation, optional demo-data choices, progress, and a final one-time completion state. It is inaccessible after setup except through controlled maintenance tooling.
- **API behavior:** Implement `getInstallationStatus`, `validateInstallationEnvironment`, `initializeControlPlane`, and `seedOptionalDemoData`. All setup commands are idempotent and guarded against concurrent initialization.
- **Data model:** Control DB: installation state/version, first Application Administrator, seed records, and setup audit events. Optional demo project data uses a separate Project DB provisioned through the normal project workflow.
- **Acceptance criteria:**
  - [ ] A clean environment can validate prerequisites, apply the Control DB schema, and create the first Application Administrator.
  - [ ] Setup is idempotent and protected against concurrent/second initialization.
  - [ ] Demo user/project seeding is optional, visibly non-production, and creates its project database through the normal isolated provisioning path.
  - [ ] No external editor, DSL file, or Tomcat dependency is used by installation.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Install the Control DB and first administrator idempotently, with optional non-production demo data.

## Proposed prerequisites

- [01 — Implement runtime config validation](01-runtime-config.md)
- [02 — Add Docker Compose local stack](02-docker-compose.md)
- [03 — Create control Prisma schema](03-control-prisma.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Implement `getInstallationStatus`, `validateInstallationEnvironment`, and `initializeControlPlane` with guards against concurrent or repeated initialization.
2. Apply the Control DB schema and create the first Application Administrator, required settings, and installation state/version.
3. Create a protected first-run screen with validation, administrator creation, progress, and one-time completion; disable public setup afterward.
4. Integrate `seedOptionalDemoData` only after the normal creation/provisioning flow in guides 16 and 18 is ready.
5. Create demo data in a separate Project DB, clearly marked as non-production, and audit setup; do not depend on an external editor, DSL, or Tomcat.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/UI behavior fields.

- [ ] `getInstallationStatus` correctly reports "not installed" on a clean environment and "installed" afterward.
- [ ] `validateInstallationEnvironment` checks real prerequisites (e.g. DB reachability, required config from [01](01-runtime-config.md)) and reports actionable failures instead of a generic error.
- [ ] `initializeControlPlane` applies the Control DB schema and creates exactly one first Application Administrator; running it twice, or concurrently, does not create a second administrator or duplicate schema application (test this explicitly).
- [ ] The first-run/setup screen is inaccessible once installation is complete, except through explicitly controlled maintenance tooling — it must not remain a public, unauthenticated endpoint after go-live.
- [ ] `seedOptionalDemoData` is optional (skippable), visibly labeled as non-production in the UI, and creates its demo project through the same isolated provisioning path as any real project (no shortcut that bypasses [16](16-project-template.md)/[18](18-protocol-wizard.md)).
- [ ] No step in installation depends on the legacy external editor, a DSL file, or a Tomcat process.
- [ ] Setup actions (schema applied, administrator created, demo seeded) are recorded as Control DB audit events.
- [ ] Automated tests cover: fresh install success; a second/concurrent install attempt being rejected or safely no-op; and demo seeding using the normal provisioning path (mocked if [16](16-project-template.md) isn't implemented yet, with that limitation explicitly reported).
- [ ] Report which commands were run and their actual results, and explicitly flag if demo seeding could not be fully tested because its prerequisites (16/18) are not yet implemented.

## Gaps and pending decisions

Run bootstrap after the Control schema and before validating login end to end; complete demo seeding after the wizard. This two-stage split is a planning proposal to avoid an artificial dependency cycle.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [08.01 — Implement getInstallationStatus, validateInstallationEnvironment, and initializeControlPlane with guards…](sub-issues/08-01.md)
- [08.02 — Apply the Control DB schema and create the first Application Administrator, required settings, and…](sub-issues/08-02.md)
- [08.03 — Create a protected first-run screen with validation, administrator creation, progress, and one-time completion](sub-issues/08-03.md)
- [08.04 — Integrate seedOptionalDemoData only after the normal creation/provisioning flow in guides 16 and 18 is ready](sub-issues/08-04.md)
- [08.05 — Create demo data in a separate Project DB, clearly marked as non-production, and audit setup](sub-issues/08-05.md)
