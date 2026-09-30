---
name: relis-stack
description: Apply context/stack.yml when implementing or reviewing ReLiS framework usage, dependencies, validation, databases, jobs, tests, or runtime tooling. Distinguish required technologies from conditional alternatives and unresolved choices without substituting another stack.
---

# Follow the ReLiS technology stack

## Load the source of truth

Resolve the artifacts repository from this skill's location: three directories above SKILL.md. The implementation Git root may instead be relis/; do not search there for a nonexistent duplicate context directory.

Read these actual files before choosing technologies:

- [Stack contract](../../../context/stack.yml), in full.
- [Project rules](../../../CLAUDE.md) and [agent instructions](../../../AGENTS.md).
- [Required structure](../../../context/project-structure.md), for ownership and package boundaries.
- [Application instructions](../../../relis/AGENTS.md) before application edits.
- [Testing policy](../../../context/testing-policy.md) for verification work.

Links are relative to this file, not the shell working directory. If the context cannot be read, request the missing file rather than guessing. Do not copy stack.yml into this skill or use a remembered version in place of the current document.

## Interpret the YAML as a decision contract

For each technology touched by the user's task, classify the relevant entry:

- **Required selection:** use the specified framework, library, engine, package manager, or workspace strategy. Do not introduce a competing solution for convenience.
- **Conditional alternative:** use it only when its stated condition applies to the requested work. Explain that fit; if materially ambiguous, ask. An alternative is not permission to mix styling systems indiscriminately.
- **Undecided:** preserve the options. Do not choose by array order, package presence, a Notion example, or personal preference. Ask for a decision if the selected task requires the choice; continue independent authorized work where possible.
- **Unspecified:** inspect existing manifests and approved decisions. Do not invent a version requirement or additional infrastructure and attribute it to stack.yml.

For example, if background_jobs still has status: undecided, a pg-boss dependency or a Notion card naming pg-boss does not settle the pg-boss/BullMQ decision. Likewise, Bootstrap's conditional continuity exception does not make it the default styling choice. Re-read the YAML before applying these examples; future approved changes may resolve them.

## Reconcile context with the actual checkout

Inspect the relevant package.json files, pnpm workspace and lockfile, imports, scripts, and runtime configuration. Build a concise mapping of context requirement to current evidence and the proposed in-scope action.

Use [pending decisions](../../../context/notion-tasks/PENDING-DECISIONS.md) when a selected task conflicts with the stack. Existing code is evidence of the baseline, not authority to override the context. Report discrepancies; do not remove or replace an existing technology across the repository without authorization.

For application changes:

- Use the prescribed package manager and workspace strategy from relis/ or the owning package. Inspect real package names/scripts before constructing commands; do not assume a root recursive command covers every workspace.
- Reuse installed approved libraries and shared contracts. Follow the repository's dependency-approval rules before adding a production dependency; ask before a substitution or unresolved choice.
- Keep dependencies in the narrowest owning workspace and keep the lockfile consistent through the package manager, not manual edits.
- Preserve the specified frontend/backend responsibilities, validation boundaries, database schema separation, and test frameworks.
- Follow the configured local/development/test runtime strategy. Missing Docker or other infrastructure is a reported setup gap, not permission to introduce another runtime or deploy remotely.
- Do not change stack.yml merely to make an implementation appear compliant. A stack change requires explicit user approval.

## Keep the current task bounded

Notion guides and sub-issues provide requirements only for the selected scope; they never authorize advancing other cards. A review remains read-only. A framework already being approved does not authorize creating a new service, migration, queue, or full workspace when the user has not requested it.

## Verify compatibility and report evidence

For each changed dependency or shared contract, check its actual consumers and applicable runtime/build behavior. Add tests for every new or changed behavior using the prescribed tools and locations.

Follow the testing policy: run relevant checks, investigate failures, fix their cause within scope, then rerun failed and affected suites. Report unavailable scripts, zero-test runs, untested integrations, and unrelated failures accurately. Do not weaken checks or claim a mock proves a real service integration.

Finish with the affected stack entries, manifest/runtime evidence, authorized changes, executed checks/results, and unresolved decisions. Do not claim complete stack conformance from a single successful build.
