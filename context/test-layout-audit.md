# Structure and test audit — 2026-09-27

## Scope and evidence

Read-only inventory of the relis submodule in vibe-coding-exp, plus its package manifests and AGENTS.md, compared with [project-structure.md](project-structure.md). This is a different checkout from earlier runtime-config implementation reports; their reported test counts are not evidence for this checkout. No application test command was run in this documentation update.

## Findings

| Area | Observed baseline | Required follow-up by Claude Code |
| --- | --- | --- |
| Test files | No tracked/source *.test.* or *.spec.* files, tests/ tree, Vitest configuration, or Playwright configuration found in the inventory (excluding dependencies/builds) | Add tests for existing behavior within separately authorized implementation scope; there are no tests here to relocate |
| Worker and contracts | Both test scripts use vitest run --passWithNoTests | Configure real discovery and ensure empty suites cannot satisfy feature acceptance |
| API, web, database | No test script in their package manifests | Wire relevant coverage without claiming pnpm -r test covers every workspace |
| API layout | apps/api/src/index.ts; no app.ts/platform/modules structure | Separate startup into main.ts and Hono assembly into app.ts; update dev/build/start paths and verify behavior when authorized |
| Web layout | Default src/app/page.tsx and layout.tsx; no localized route/feature/shared structure | Plan scoped alignment with src/app/[locale], features, shared, and i18n; localization behavior is not a mere file move and requires agreed scope |
| Worker layout | src/main.ts exists, no platform/jobs separation yet | Introduce boundaries only as actual responsibilities are implemented |
| Database | Package exists; no control/project Prisma schemas in the inventory | Do not invent migrations as part of a test relocation task |
| Agent instructions | Previous target tree conflicted with mandatory context (web components/features, API routes/services/index, worker queues/services) | Instruction drift corrected in this update; application layout has not been changed |
| Queue choice | pg-boss dependency exists, while stack.yml leaves pg-boss/BullMQ undecided | Request an explicit decision when queue implementation is selected; package presence is not approval |

## Canonical test organization

Integration, end-to-end, migration/parity, and fixtures belong under relis/tests/ in their existing prescribed categories. Helpers belong in relis/packages/test-utils/. Unit-test placement was not specified in the supplied tree and remains a recorded placement decision; see [testing-policy.md](testing-policy.md).

No directory tree should be populated just for appearance. Missing future modules are not defects requiring immediate scaffolding.

## Handoff

Use [the scoped restructuring prompt](prompts/align-structure-and-tests.md) with Claude Code when authorizing implementation. Re-inventory first: this audit is a dated snapshot, not a continuing guarantee.

Acceptance for the follow-up: existing behavior preserved, imports/scripts/discovery updated, real tests execute in the approved locations, failure cases covered, relevant checks rerun, and no unrelated Notion functionality added.
