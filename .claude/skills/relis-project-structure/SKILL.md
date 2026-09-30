---
name: relis-project-structure
description: Apply the ReLiS repository structure from context/project-structure.md when planning, implementing, moving, or reviewing application files, module boundaries, and test locations. Use for architecture conformance, not to scaffold unrequested features.
---

# Follow the ReLiS project structure

## Read the maintained context

Locate the artifacts repository containing this skill. From this SKILL.md, its root is three directories up; do not assume the shell's current Git root is that repository, because relis/ is a separate Git submodule.

Before making placement decisions, read:

- [Project rules](../../../CLAUDE.md) and [agent instructions](../../../AGENTS.md).
- [Required project structure](../../../context/project-structure.md), in full.
- [Technology stack](../../../context/stack.yml) for the responsibilities attached to that structure.
- [Application instructions](../../../relis/AGENTS.md) before changing application files.
- [Testing policy](../../../context/testing-policy.md) when placing or changing tests.

These links are relative to this skill file, not the terminal working directory. Read the actual files; mentioning a link does not load it. If a required reference is unavailable, report the missing path and request it instead of reconstructing the architecture from memory.

The maintained context is authoritative. This skill explains how to apply it; it is not a second copy of the directory tree.

## Map the authorized change

1. Identify whether the request is implementation, review, or explanation. Review/explanation remains read-only. Loading this skill does not authorize implementation.
2. Inspect both repositories' working-tree status and the relevant existing files. Distinguish required target structure from current implementation; a path in the tree does not prove the feature exists.
3. For each proposed addition or move, identify its responsibility, owning application/package, required target path, and affected imports/consumers. For a move, show the old-to-new mapping before editing.
4. Keep all application code, tests, runtime configuration, and application tooling inside relis/. Keep context and session artifacts in the parent. Never create relis/relis/ or a sibling relis_next/ checkout.
5. Create only paths needed for the selected task. Missing future modules do not authorize a full scaffold or a repository-wide refactor.

## Apply responsibility boundaries

Use the current structure document to resolve exact paths. In particular:

- Web route composition belongs in the prescribed App Router tree; domain behavior belongs in features, and genuinely reusable primitives in shared. Do not replace these boundaries with a parallel components/features architecture.
- API startup, application assembly, platform services, and business modules have distinct responsibilities. Do not recreate an alternative top-level routes/services/repositories architecture because a scaffold or example uses it.
- Worker startup, worker-safe platform services, and job handlers remain distinct. Reuse shared contracts without importing application internals into shared packages.
- Preserve the control/project database schema and client boundaries. Browser code must not access Prisma or database credentials.
- Put integration, end-to-end, migration/parity tests, fixtures, and shared helpers in their prescribed locations. Follow the testing policy for unit-test placement; an unspecified location is not permission to invent a required directory.

If the existing implementation deviates, correct only the part required by the authorized task. Report unrelated deviations without silently migrating them. Do not rewrite project-structure.md to justify a convenient implementation.

## Use task references without expanding scope

When a Notion task is selected, read [the task index](../../../context/notion-tasks/README.md), that parent/sub-issue, and relevant entries in [pending decisions](../../../context/notion-tasks/PENDING-DECISIONS.md). Do not load or implement the entire backlog. Source contradictions and missing dependencies require an explicit decision when they block this task, not automatic implementation of another issue.

## Verify and hand off

For changed paths, check imports, exports, startup scripts, TypeScript configuration, and test discovery. Run the checks required by the testing policy; diagnose failures, fix their cause within scope, and rerun affected checks. Never treat an empty test run as coverage.

Report the resulting placement and responsibility mapping, tests actually run, remaining deviations, and any decision needed. For review-only requests, report findings with file evidence and do not modify files. Do not commit, update the submodule revision, or archive sessions unless requested.
