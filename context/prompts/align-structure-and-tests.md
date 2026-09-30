# Claude Code prompt — align existing structure and tests

Use this prompt only when authorizing implementation; it is not an instruction to run automatically.

Work only inside relis/, following its AGENTS.md, parent CLAUDE.md, context/project-structure.md, context/stack.yml, context/testing-policy.md, and context/test-layout-audit.md.

Re-inventory this checkout. Align existing code and tests with the prescribed structure, preserving behavior. Do not implement the backlog, add missing business modules/schemas, select a queue technology, or scaffold empty directories.

1. Present an old-path → new-path mapping for existing files. API uses main.ts, app.ts, platform/, and modules/. Web uses app/[locale]/, features/, shared/, and i18n/. Worker uses main.ts, platform/, and jobs/. Move only existing responsibilities; request approval if alignment changes behavior beyond restructuring.
2. Inventory tests and scripts. If tests are absent, say so and add regression/smoke coverage for existing behavior rather than claiming to move nonexistent tests.
3. Put integration tests in tests/integration/, Playwright journeys in tests/e2e/, migration/parity checks in tests/migration/, fixtures in tests/fixtures/, and helpers in packages/test-utils/, relative to relis. Unit-test placement is unspecified: propose a minimal convention and obtain a decision before introducing a location.
4. Update imports, exports, commands, TypeScript configuration, and discovery so relocated code and root-level tests actually run. Check test code without shipping it. Do not use --passWithNoTests or skipped workspaces as proof of coverage.
5. Capture baseline behavior with tests, restructure, and run affected suites plus available typecheck/lint/build. Diagnose every failure, fix its cause, and rerun failed and affected checks. Report blockers rather than weakening checks or expanding scope.
6. Report moves, preserved behavior, test locations, discovered/executed counts, exact commands/results, and deviations. Inspect both repositories; do not commit, update the submodule pointer, deploy, or touch production.

Do not claim complete architecture conformance if only a subset was authorized. Do not claim real infrastructure integration from mocks.
