# 17 — Implement project database resolver

## Source and usage

- Source: [Implement project database resolver](https://app.notion.com/p/Implement-project-database-resolver-3b44993602938053a925e177bce47124).
- Notion ID: `3b44993602938053a925e177bce47124`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Implement the server-only resolver inside the existing database/platform package structure. `@relis/database` currently has no client or resolver implementation; this work maps a verified project ID to its ready, isolated PostgreSQL Prisma client.
- **Functional Domain:** Infrastructure
- **Behavior:** Empty
- **Users and permissions:** Empty
- **UI behavior:** No direct UI. Project screens show a safe unavailable/provisioning state when the resolver reports a non-ready workspace.
- **API behavior:** Implement `resolveProjectDatabaseClient`, `getProjectWorkspaceStatus`, and `invalidateProjectClientCache`. Inputs are internal verified project IDs, never client-supplied database names/URLs.
- **Data model:** Empty
- **Acceptance criteria:** Empty
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Resolve a verified, ready project's Prisma client on the server.

## Proposed prerequisites

- [04 — Implement reusable project Prisma schema](04-project-prisma.md)
- [15 — Implement Project Context, Roles, and Membership](15-membership.md)
- [16 — Prepare and Provision an Isolated Project Database Template](16-project-template.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Locate the existing database/platform package and the Control DB project registry.
2. Implement `resolveProjectDatabaseClient` using only an internal, previously authorized `projectId`.
3. Verify readiness and compatibility before accessing the Project DB.
4. Implement `getProjectWorkspaceStatus` and `invalidateProjectClientCache`.
5. Define client lifecycle/cache behavior and return safe unavailability when the workspace is not ready.

## Definition of Done — completion checklist

Notion's Acceptance criteria property is empty; the checklist below combines that fact with concrete points grounded in the API behavior/UI behavior fields above.

- [ ] `resolveProjectDatabaseClient` accepts only an internal, already-verified `projectId` — it never accepts a client-supplied database name, URL, or connection string as input.
- [ ] A project that is not `Ready` (still `Provisioning` or `Failed` per [16](16-project-template.md)) resolves to a safe "unavailable" status rather than throwing an unhandled error or returning a broken client.
- [ ] `getProjectWorkspaceStatus` lets calling code (route handlers) check readiness before attempting real work, so project screens can show a safe unavailable/provisioning state instead of a raw error.
- [ ] `invalidateProjectClientCache` exists and is actually called when a project's state changes in a way that invalidates a cached client (e.g., after successful provisioning) — verified with a test that a stale "not ready" resolution doesn't linger after readiness changes.
- [ ] Clients resolved for two different projects never share a connection/state that would let data leak between them (a resolved client for project A must not be reusable to query project B).
- [ ] Resolved clients follow a defined lifecycle (created lazily, cached, eventually closed) rather than instantiating a new `PrismaClient` per call, consistent with [AGENTS.md](../../relis/AGENTS.md)'s "reuse clients" rule.
- [ ] Automated tests cover: resolving a ready project's client and running a trivial query; resolving a non-ready project and getting the safe unavailable status; and cache invalidation actually taking effect.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

Data model, permissions, and acceptance criteria are empty in the source. The exact cache strategy is a technical decision; the checks below are proposed.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [17.01 — Locate the existing database/platform package and the Control DB project registry](sub-issues/17-01.md)
- [17.02 — Implement resolveProjectDatabaseClient using only an internal, previously authorized projectId](sub-issues/17-02.md)
- [17.03 — Verify readiness and compatibility before accessing the Project DB](sub-issues/17-03.md)
- [17.04 — Implement getProjectWorkspaceStatus and invalidateProjectClientCache](sub-issues/17-04.md)
- [17.05 — Define client lifecycle/cache behavior and return safe unavailability when the workspace is not ready](sub-issues/17-05.md)
