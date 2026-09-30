# 02 — Add Docker Compose local stack

## Source and usage

- Source: [Add Docker Compose local stack](https://app.notion.com/p/Add-Docker-Compose-local-stack-3b449936029380bcbd8be3437ececdf5).
- Notion ID: `3b449936029380bcbd8be3437ececdf5`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide one-command local development with the ReLiS2.0 web app, API, worker, PostgreSQL, reverse proxy, local object storage, and local mail capture.
- **Functional Domain:** Infrastructure
- **Behavior:**
  - `deployment/docker-compose.yml` starts MariaDB, phpMyAdmin, Tomcat, Bibler, and the PHP application.
  - The legacy stack mounts source code and Tomcat workspace data, including the DSL/editor generation flow.
- **Users and permissions:** Empty
- **UI behavior:** No product feature UI. Local MailHog/Mailpit and job observability endpoints are documented for development use only.
- **API behavior:** Expose health/readiness endpoints only through the local reverse proxy; bind database and storage ports safely for development.
- **Data model:** Local persistent volumes hold the Control DB, project test databases, and non-production object-storage artifacts. They are disposable and excluded from source control.
- **Acceptance criteria:** Empty
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Provide one-command local development with web, API, worker, PostgreSQL, reverse proxy, object storage, and mail capture.

## Proposed prerequisites

- [01 — Implement runtime config validation](01-runtime-config.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Check which applications exist and their actual commands; do not assume the monorepo described in Notion already exists in this checkout.
2. Define services and shared configuration, including the Control DB and per-project test databases.
3. Configure local persistent volumes and exclude them from version control.
4. Expose health/readiness through the local proxy and restrict database and storage port exposure.
5. Document startup, diagnostics, and local access to MailHog/Mailpit and job observability.

## Definition of Done — completion checklist

Notion's Acceptance criteria property is empty; the checklist below combines that fact with concrete points grounded in the Description/Behavior/UI/API/Data model fields above.

- [ ] A single documented command (e.g. `docker compose up`) starts web, API, worker, PostgreSQL, a reverse proxy, local object storage, and local mail capture, and all become ready.
- [ ] The Control DB and per-project test database(s) are provisioned as separate, disposable local volumes, excluded from source control (`.gitignore`).
- [ ] Health/readiness endpoints are reachable only through the local reverse proxy, not by directly exposing every internal service port.
- [ ] Database and storage ports are bound safely for local development (not indiscriminately exposed on all interfaces).
- [ ] MailHog/Mailpit (or equivalent) and job-observability endpoints are documented as development-only; they are not presented as product features in any UI.
- [ ] No local credentials, generated volumes, or `.env` files with real values are committed.
- [ ] Startup and diagnostics steps are documented (README or docs/) so another developer can reproduce the stack from a clean checkout.
- [ ] Report which commands were actually run to validate the stack (e.g., bringing it up, checking each service's health) and any service that could not be verified in this environment.

## Gaps and pending decisions

Acceptance criteria are not provided in the source. BiBler is required by the import task (29) but is not included in this task's new service list; integrate or document its availability before implementing that import.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [02.01 — Check which applications exist and their actual commands](sub-issues/02-01.md)
- [02.02 — Define services and shared configuration, including the Control DB and per-project test databases](sub-issues/02-02.md)
- [02.03 — Configure local persistent volumes and exclude them from version control](sub-issues/02-03.md)
- [02.04 — Expose health/readiness through the local proxy and restrict database and storage port exposure](sub-issues/02-04.md)
- [02.05 — Document startup, diagnostics, and local access to MailHog/Mailpit and job observability](sub-issues/02-05.md)
