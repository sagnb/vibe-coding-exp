# 04 — Implement reusable project Prisma schema

## Source and usage

- Source: [Implement reusable project Prisma schema](https://app.notion.com/p/Implement-reusable-project-Prisma-schema-3b4499360293809895dbd69c461aa143).
- Notion ID: `3b4499360293809895dbd69c461aa143`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Extend the existing `@relis/database` package with the reusable, versioned PostgreSQL schema applied to every isolated project database. No Prisma schema or migration exists yet. Dynamic protocol behaviour is represented as metadata and versioned records, never generated application code, ad hoc tables, or stored procedures.
- **Functional Domain:** Infrastructure
- **Behavior:**
  - Legacy `Install.php` installs common/project-specific tables, views, and stored procedures from generated configuration.
  - `Config.php` and the legacy editor pipeline generate classification/extraction structures per project.
- **Users and permissions:** Empty
- **UI behavior:** Empty
- **API behavior:** Provide typed project-runtime repositories/contracts for papers, protocol metadata, workflow assignments, screening, QA, extraction, reporting metadata, local audit references, and schema version checks.
- **Data model:** Empty
- **Acceptance criteria:** Empty
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Create one fixed, reusable PostgreSQL schema for each isolated project database; represent dynamic protocol behavior as metadata.

## Proposed prerequisites

- [03 — Create control Prisma schema](03-control-prisma.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Inventory the Paper, protocol, assignment, screening, QA, classification, reporting, and audit contracts described in the guides.
2. Extend the existing database package with a schema and static migrations separate from the Control DB.
3. Represent categories, dependencies, and subcategories as records in fixed models, without generating tables per category.
4. Create typed repositories and schema version/compatibility checks.
5. Validate the same schema in two independent databases and prepare it for the provisioning template.

## Definition of Done — completion checklist

Notion's Acceptance criteria property is empty; the checklist below combines that fact with concrete points grounded in the Description/Behavior/API behavior fields above.

- [ ] Exactly one fixed, versioned Prisma schema exists for project databases; it is applied identically to every project — no per-project or per-category generated tables, views, or stored procedures.
- [ ] Protocol-specific behavior (extraction categories, dependencies, subcategories, screening/QA/classification configuration) is represented as configuration rows in fixed tables, never as dynamically generated schema.
- [ ] Two independently provisioned project databases using this schema do not share or leak each other's data (verified with a test against two seeded instances).
- [ ] Typed repositories/contracts are exposed for papers, protocol metadata, workflow assignments, screening, QA, extraction, reporting metadata, and local audit references — no generic/raw Prisma client is exposed to consumers.
- [ ] A schema-version/compatibility check exists so the application can detect a project database that doesn't match the currently expected fixed schema.
- [ ] Migrations apply cleanly to a fresh database, and this schema is what [16 — Prepare and Provision an Isolated Project Database Template](16-project-template.md) will clone as `relis_project_template`.
- [ ] Automated tests cover at least: migration application, one repository operation per major entity group, and the schema-version check.
- [ ] Report which commands were run and their actual results, including anything not testable in this environment.

## Gaps and pending decisions

Data model and acceptance criteria are empty in the source. The exact design depends on domain contracts; do not invent final columns from this summary alone.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [04.01 — Inventory the Paper, protocol, assignment, screening, QA, classification, reporting, and audit contracts…](sub-issues/04-01.md)
- [04.02 — Extend the existing database package with a schema and static migrations separate from the Control DB](sub-issues/04-02.md)
- [04.03 — Represent categories, dependencies, and subcategories as records in fixed models, without generating…](sub-issues/04-03.md)
- [04.04 — Create typed repositories and schema version/compatibility checks](sub-issues/04-04.md)
- [04.05 — Validate the same schema in two independent databases and prepare it for the provisioning template](sub-issues/04-05.md)
