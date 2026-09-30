# Mandatory testing and verification policy

Applies to application work inside relis/. Read the [required structure](project-structure.md) and [stack](stack.yml) first. Vitest is the unit/integration framework; Playwright is the end-to-end framework. This policy does not authorize unrelated features or a full infrastructure scaffold.

## Feature and bug-fix workflow

1. Read the selected parent/sub-issue, actual implementation, and relevant source contradictions. Map each applicable acceptance criterion to an automated test with an observable expected result.
2. Add tests for every new feature or changed behavior. Cover successful use, invalid input, forbidden operations, edge cases, and project/user isolation when applicable. For a bug, reproduce the original failure with a regression test before fixing it where practical.
3. Implement the authorized change and run focused tests. On any failure, inspect the output, determine whether the defect is in product code, test assumptions, or environment, fix the actual cause, and rerun.
4. After focused tests pass, rerun all affected suites and consumers plus available typecheck, lint, and build checks. Structural moves require test-discovery and import-resolution checks as well as behavior checks.
5. Repeat diagnosis, correction, and verification until passing. Do not silently ignore a pre-existing or unrelated failure: report its evidence and request scope approval if fixing it is outside the task. Missing infrastructure or permission is a blocker, not a passing result.
6. Finish with criterion-to-test traceability, exact commands and working directories, exit codes, counts of passed/failed/skipped tests, and remaining limitations. Do not mark the task complete if required checks were not performed or required integrations remain absent.

Never delete, skip, quarantine, weaken assertions, increase timeouts without diagnosis, disable validation, or change production behavior merely to make tests pass. A deliberate specification change needs explicit approval and corresponding test updates.

## Test locations

Paths below are relative to the relis/ application root, not the artifacts repository.

| Kind | Required location | Responsibility |
| --- | --- | --- |
| Integration | tests/integration/<feature>/ | API, persistence, process startup, worker and service-boundary behavior |
| End-to-end | tests/e2e/<feature>/ | User journeys through real application boundaries using Playwright |
| Migration/parity | tests/migration/<feature>/ | Schema upgrades, legacy fixtures, data preservation and migration compatibility |
| Shared fixtures | tests/fixtures/<feature>/ | Synthetic, deterministic input/expected data; no credentials or personal records |
| Shared helpers | packages/test-utils/ | Reusable setup, builders, isolation and cleanup utilities |

The supplied structure does **not** specify a dedicated unit-test location. Do not misclassify unit tests as integration tests or invent tests/unit as an approved architecture requirement. Preserve an existing unit-test convention if found; otherwise obtain a placement decision before adding the first unit suite. This ambiguity does not prevent adding correctly located integration tests or documenting unit cases.

Configure discovery explicitly so root-level tests run from the application workspace. Avoid duplicate execution by multiple package runners. Keep test-only files out of production output, without dropping them from type checking. Do not create empty folders for features that do not exist.

## Evidence quality and safe execution

- A successful command with zero discovered tests is not feature coverage. In particular, --passWithNoTests must not be used as evidence that a feature is tested.
- Mocks prove the mocked boundary contract, not real database, process, mail, storage, queue, or browser integration. Label those results accurately; use disposable local infrastructure for integration evidence when available and authorized.
- Startup tests must assert ordering: invalid configuration exits nonzero before listening, connecting, or writing. Test the actual entry command when making a startup guarantee.
- For processes, assert exit codes, observable state, and cleanup of children; a startup banner or a timeout alone is not proof of success. Avoid fixed sleeps where observable readiness is available.
- Use isolated disposable databases/projects/files; restore state and close servers, clients, and child processes in cleanup even on failure. Never use production or an unspecified database for tests.
- Check authorization at the API boundary, not merely hidden buttons. Include cross-project access rejection where applicable.
- Use deterministic data and controlled clocks/randomness. Use actual concurrency tests for race-sensitive invariants when feasible.
- Report missing scripts as missing. A successful recursive command may skip packages without a matching script; list which packages and suites were actually covered.
- Do not install dependencies, launch external services, run migrations, or deploy outside the user's authorized scope.

## Documentation-only changes

Validate links, issue IDs, step coverage, source attribution, and consistency of instructions. Report that application tests were not run when application code was not changed. Documentation existence does not prove feature completion.
