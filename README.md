# Vibe-coding experiment

This repository stores the artifacts used to guide and review implementation.
Application code belongs to the separate `relis_next` repository, mounted here as the `relis/` Git submodule.

- `context/project-structure.md`: required application repository structure, rooted at `relis/`.
- `context/stack.yml`: project technology stack.
- [Testing policy](context/testing-policy.md): mandatory coverage, test placement, failure diagnosis, correction, and reruns.
- [Detailed local sub-issues](context/notion-tasks/sub-issues/README.md): one scoped specification per original guide step, with proposed acceptance and test scenarios.
- [Structure/test audit](context/test-layout-audit.md) and [Claude restructuring prompt](context/prompts/align-structure-and-tests.md): dated findings and a separately authorized implementation handoff.
- `context/notion-tasks/`: task reference material, not authorization to implement unrequested features.
- `claude-sessions/`: archive Claude Code sessions that implement features in `relis/`, one session per issue. This directory is initially empty; Git does not track empty directories.
- `relis/`: application implementation submodule. Commit application changes in that repository and update its submodule reference here when appropriate.

Initialize the implementation checkout after cloning:

```sh
git submodule update --init --recursive
```

Keep session records free of credentials and sensitive data before committing or sharing them.
