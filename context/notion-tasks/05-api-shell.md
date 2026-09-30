# 05 — Implement API app shell

## Source and usage

- Source: [Implement API app shell](https://app.notion.com/p/Implement-API-app-shell-3b449936029380fa908becac920f75cf).
- Notion ID: `3b449936029380fa908becac920f75cf`.
- Source domain: Infrastructure.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** `relis-next` already has a Hono server with CORS, `GET /`, and `GET /health`. Complete that baseline with route registration, typed contracts, request IDs, authentication/policy/context hooks, stable error mapping, validation, readiness, and structured observability.
- **Functional Domain:** Infrastructure
- **Behavior:**
  - Legacy `Api.php` returns JSON protocol/report data by project label and opens the selected project's database directly.
  - Legacy errors, authorization, and project selection are controller/session driven rather than a shared typed API contract.
- **Users and permissions:** The React app receives consistent localized error states. Operational health endpoints are separate from end-user screens.
- **UI behavior:** The React app receives consistent localized error states. Operational health endpoints are separate from end-user screens.
- **API behavior:** Implement `health`, `readiness`, request-ID middleware, schema validation, error mapper, API version/route registration, authentication hook, project-context hook, and OpenAPI/contract generation where selected. Return stable codes for validation, authentication, authorization, not-found, conflict, and internal errors.
- **Data model:** Empty
- **Acceptance criteria:** Empty
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Complete the Hono server with routes, typed contracts, validation, stable errors, readiness, and observability.

## Proposed prerequisites

- [01 — Implement runtime config validation](01-runtime-config.md)
- [03 — Create control Prisma schema](03-control-prisma.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Verify the stated baseline: Hono, CORS, `GET /`, and `GET /health`.
2. Organize route registration/versioning, contracts, and input validation.
3. Add request IDs, structured logs, and error mapping for validation, authentication, authorization, not found, conflict, and internal errors.
4. Define authentication, policy, and project-context integration points; integrate real implementations in their respective tasks.
5. Separate health from readiness; include OpenAPI/contract generation only if selected for the project.

## Definition of Done — completion checklist

Notion's Acceptance criteria property is empty; the checklist below combines that fact with concrete points grounded in the Description/API behavior fields above.

- [ ] `GET /health` (liveness) and a separate `GET /readiness` (dependency-aware) endpoint both exist and behave differently — readiness reflects actual dependency state (DB, etc.), health does not.
- [ ] Every request carries a request ID (generated or propagated), surfaced in logs and, where useful, in error responses.
- [ ] All request bodies/params/query strings that reach a handler are schema-validated (Zod, per the stack) before business logic runs.
- [ ] A single error mapper returns stable, documented codes/shapes for validation, authentication, authorization, not-found, conflict, and internal errors — no raw stack traces or internal messages leak to the client.
- [ ] Route registration is versioned/organized so new feature modules can register routes without editing a monolithic file; `src/main.ts` stays limited to startup/shutdown per [AGENTS.md](../../relis/AGENTS.md).
- [ ] Authentication and project-context hooks exist as integration points (middleware) even before their concrete implementations ([09](09-login-sessions.md), [15](15-membership.md)) land — they must not silently no-op and leave routes unintentionally public.
- [ ] Structured (JSON) logs are emitted with request ID and route context; no secrets or full payloads are logged.
- [ ] Automated tests cover: health vs readiness distinction, at least one validation-failure case, and the error-mapper's shape for each error class listed above.
- [ ] Report which commands were run (typecheck, tests, manual `curl` checks) and their actual results.

## Gaps and pending decisions

Data model and acceptance criteria are empty. Users and permissions repeats UI-related text and therefore does not provide a permission matrix.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [05.01 — Verify the stated baseline: Hono, CORS, GET /, and GET /health](sub-issues/05-01.md)
- [05.02 — Organize route registration/versioning, contracts, and input validation](sub-issues/05-02.md)
- [05.03 — Add request IDs, structured logs, and error mapping for validation, authentication, authorization, not…](sub-issues/05-03.md)
- [05.04 — Define authentication, policy, and project-context integration points](sub-issues/05-04.md)
- [05.05 — Separate health from readiness](sub-issues/05-05.md)
