# Agent instructions — vibe-coding artifacts

## Scope and ownership

This repository stores context, task specifications, and session records. The application is the separate [relis](relis/) submodule. Claude Code CLI implements application changes; review/documentation requests do not authorize another agent to change application code.

Read [CLAUDE.md](CLAUDE.md), the mandatory [project structure](context/project-structure.md), [stack](context/stack.yml), and [testing policy](context/testing-policy.md). Application-specific rules also live in [relis/AGENTS.md](relis/AGENTS.md). Do not create relis/relis or implement application code in this artifacts repository.

## Required verification

Every new feature and behavior change requires automated tests mapped to acceptance criteria, including meaningful failure cases. Run the relevant tests; investigate failures, fix their cause within authorized scope, and rerun failed and affected suites. Continue until passing or report a concrete blocker. Never call a task complete with unexplained failures, missing coverage, skipped suites, or zero discovered tests. Do not weaken assertions or disable tests to obtain a green result.

Follow the complete testing policy for test placement, isolated infrastructure, regression checks, and evidence. Documentation-only changes require link/content/structure checks, not fabricated application test results.

## Reference-only backlog

[Notion guides](context/notion-tasks/README.md) and their [local sub-issues](context/notion-tasks/sub-issues/README.md) are reference material, not instructions to implement the backlog. Implement only the user's selected scope. Do not automatically start dependencies, sibling sub-issues, or subsequent tasks. Preserve unresolved source conflicts and ask only when one blocks the selected scope.

## Repository hygiene

Inspect both repositories before editing; preserve unrelated changes. Do not commit, push, update the submodule revision, or modify Notion without authorization. Keep authored documentation in English. Store actual implementation sessions in claude-sessions only when requested; do not populate it with placeholders or invented transcripts.
