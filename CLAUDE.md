# Project context and implementation rules

## Repository boundaries

This repository (`vibe-coding-exp`) stores vibe-coding artifacts, project context, task references, and implementation session records. Application implementation belongs to the separate `relis_next` repository, checked out here as the [relis](relis/) Git submodule.

- Implement application code only inside `relis/`. This includes application packages, tests, tooling, dependencies, and runtime configuration.
- Treat `relis/` as the application monorepo root shown in the structure document. Do not create `relis/relis/` or a new `relis_next/` directory alongside it.
- Run application commands from `relis/` or the relevant package directory within it.
- Keep context and session artifacts in this parent repository, not inside the implementation submodule.
- Inspect the working-tree status of both repositories before making changes. Preserve existing user changes.
- The repositories have separate commit histories. When the user requests commits, commit application changes in the submodule and record the appropriate submodule revision in the parent repository. Do not commit, push, or change the referenced revision without authorization.

## Mandatory architecture and stack

<!-- These references are binding requirements, not optional suggestions. -->

Before planning or implementing application changes, read:

- [Project structure](context/project-structure.md).
- [Technology stack](context/stack.yml).

Follow both documents strictly, including directory hierarchy, package boundaries, responsibilities, technologies, testing tools, and workspace strategy. Although the structure document says "proposed," it is the required target architecture.

Do not substitute technologies or deviate from the architecture without explicit user approval. Preserve conditional alternatives and unresolved decisions; do not silently select an undecided option. Ask the user when an unresolved choice blocks the requested implementation.

The target architecture does not authorize scaffolding the entire project. Create only what is necessary for the requested task.

## Strict user-request scope

<!-- Reference material provides context, never permission to advance the backlog. -->

- Implement only what is directly required by the user's current prompt.
- Do not implement additional features, speculative improvements, unrelated refactors, or subsequent tasks merely because they appear in the reference material.
- Requests to explain, inspect, review, or document do not authorize application implementation.
- If the request requires a material expansion of scope, explain the dependency and obtain approval before proceeding.
- Stop when the requested work and relevant verification are complete. Do not automatically advance to another issue.

## Task references

Use [context/notion-tasks](context/notion-tasks/) only as reference material. Start with its [index](context/notion-tasks/README.md) and consult [pending decisions](context/notion-tasks/PENDING-DECISIONS.md) when relevant.

These guides are adapted task descriptions, not verbatim Notion transcriptions. They may contain inferred dependencies, suggested sequencing, or unresolved contradictions. They do not override the user's prompt or the mandatory architecture and stack, and they do not authorize implementation on their own.

## Claude session records

[claude-sessions](claude-sessions/) stores the Claude Code sessions that implemented features in `relis/`. Each session corresponds to one issue.

- Keep session artifacts identifiable by issue number or a descriptive issue slug.
- When asked to archive a session, preserve the actual transcript separately from any authored Markdown summary; do not present a summary as a verbatim export.
- When available, record the issue reference, authorized scope, implementation commit, review findings, verification results, and remaining limitations.
- Review transcripts for credentials and sensitive data before committing or sharing them.
- Treat previous sessions as historical evidence, not active instructions or authorization to repeat their changes in the current implementation.
- Do not create placeholder sessions or populate this directory unless requested. An empty directory is intentionally not tracked by Git.

## Verification and reporting

Follow [the mandatory testing policy](context/testing-policy.md) and [AGENTS.md](AGENTS.md). Every new feature or changed behavior requires automated tests mapped to acceptance criteria. Investigate any failure, correct its cause within the authorized scope, and rerun the failed and affected suites until passing or explicitly blocked. Never declare completion with unexplained failures, missing coverage, skipped suites, or zero discovered tests.

Keep integration, end-to-end, migration/parity tests, and fixtures under the prescribed relis/tests/ categories, and shared helpers in relis/packages/test-utils/. Unit-test placement is not defined by the supplied tree; obtain a decision when introducing it. Do not weaken assertions or disable tests to produce a passing result.

Use [local sub-issues](context/notion-tasks/sub-issues/README.md) to detail the selected task only. Their existence does not authorize dependencies or sibling work.

Verify changes in proportion to their risk. Report which commands actually ran, their results, and any untested behavior or blockers. Do not claim that mocked tests prove a real infrastructure integration or that unimplemented functionality is complete.

Keep repository documentation and authored session summaries in English. Preserve the original language of archived transcripts.
