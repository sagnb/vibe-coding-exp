# 24 — Implement Project Paper List

## Source and usage

- Source: [Implement Project Paper List](https://app.notion.com/p/Implement-Project-Paper-List-3c149936029380ad8415e359d144b867).
- Notion ID: `3c149936029380ad8415e359d144b867`.
- Source domain: Paper Library & Import.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Provide the Paper Library > Papers page for the current project, including paper search, status filters, and an Import papers entry point.
- **Functional Domain:** Paper Library & Import
- **Behavior:** Empty
- **Users and permissions:**
  - Project members authorised to access the Paper Library can view the current project's paper list.
  - Only the Project Owner and Project Managers can see and use the Import papers button.
- **UI behavior:**
  - Within the project workspace, provide a Paper library > Papers page. Display a searchable, paginated table containing papers from the current project only: Paper key | Title | URL | Source | Decision | Actions.
  - Paper key: project-local paper reference. Title: clickable, opens the Paper details page. URL: paper URL when available, otherwise `—`. Source: assigned Paper Source when available, otherwise `—`. Decision: current paper decision (Pending, Included, or Excluded). Actions: Edit paper; Delete paper — displayed only to the Project Owner and Project Managers.
  - Provide a search field supporting paper key, title, author, DOI, URL, and source. Provide filters for Pending; Included; Excluded.
  - **Paper details page** — clicking a paper title opens a read-only Paper details page displaying the available reference metadata: paper key, title, authors, URL, DOI, source, search strategy, venue, year, abstract.
- **API behavior:**
  - Use Hono as the authenticated HTTP/API layer. Enforce the current project context and project membership for every request. A paper must always be retrieved from the current Project DB — a user must never access a paper belonging to another project.
  - Provide read operations to: retrieve a paginated Paper list for the current project; search papers by paper key, title, author, DOI, URL, Paper Source; filter papers by current decision (Pending — default value; Included; Excluded); retrieve the read-only details of one paper.
  - The Paper-list response contains only the data required by the table: paper ID; paper key; title; URL; Paper Source; current decision; available actions for the authenticated user.
  - The Paper-details response contains available reference metadata: paper key; title; ordered authors; URL; DOI; Paper Source; Search Strategy; venue; publication year; abstract. Return pagination metadata with every Paper-list response.
- **Data model:** Empty
- **Acceptance criteria:** Empty
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Create the current project's Paper list and read-only details.

## Proposed prerequisites

- [04 — Implement reusable project Prisma schema](04-project-prisma.md)
- [15 — Implement Project Context, Roles, and Membership](15-membership.md)
- [17 — Implement project database resolver](17-project-resolver.md)
- [20 — Implement Project Workspace Navigation and Home Dashboard](20-project-workspace.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Implement queries against the authorized Project DB, with pagination and search by key, title, author, DOI, URL, and Source.
2. Filter by Pending (default), Included, or Excluded decisions and return pagination metadata.
3. Expose only id/key/title/URL/Source/decision and allowed actions in the list.
4. Create the Paper key | Title | URL | Source | Decision | Actions table; titles open details.
5. Show ordered authors, DOI, URL, Source, Strategy, Venue, year, and abstract in details.
6. Expose Import/Edit/Delete only to Owner/Manager when their respective flows exist.

## Definition of Done — completion checklist

Notion's Acceptance criteria property is empty; the checklist below combines that fact with concrete points grounded in the UI behavior/API behavior fields above.

- [ ] The Papers list and details endpoints resolve strictly against the caller's current project's database ([17](17-project-resolver.md)) — a paper ID from another project can never be fetched through this feature, even by direct ID guessing.
- [ ] The list table shows exactly Paper key | Title | URL | Source | Decision | Actions, with URL/Source displaying `—` when unset, and Decision defaulting to `Pending` when no filter is applied.
- [ ] Search covers paper key, title, author, DOI, URL, and Paper Source in one field; filters for Pending/Included/Excluded compose correctly with search and pagination.
- [ ] The Paper-list API response is limited to exactly: paper ID, paper key, title, URL, Paper Source, current decision, and the actions available to the authenticated caller — it does not leak full paper metadata (abstract, authors, etc.) that belongs only to the details response.
- [ ] Every list response includes pagination metadata (total count, page, page size, or cursor — pick one convention and apply it consistently across the codebase).
- [ ] Clicking a title opens a **read-only** Paper details page showing key, title, ordered authors, URL, DOI, source, search strategy, venue, year, and abstract — this page has no edit controls.
- [ ] "Edit paper" and "Delete paper" actions/buttons and their underlying endpoints are visible/reachable only to the Project Owner and Project Managers — but see Gaps below, since this ticket's own API behavior only specifies read operations.
- [ ] The "Import papers" entry point is visible only to the Project Owner and Project Managers, consistent with [20 — Project Workspace](20-project-workspace.md).
- [ ] Automated tests cover: cross-project paper access rejection; each filter (Pending/Included/Excluded) returning the correct subset; search matching on each documented field; and role-gating of Edit/Delete/Import actions.
- [ ] Report which commands were run and their actual results, and explicitly flag that Edit/Delete paper have no corresponding write API defined anywhere in the 42 Notion cards (see Gaps) — do not silently invent one.

## Gaps and pending decisions

Data model and acceptance criteria are empty; checks are proposed. The UI presents Edit/Delete actions, but the API behavior specifies only read operations for this ticket, and no separate "Paper editing/deletion" task exists among the 42 consulted Notion pages (see the [notion-tasks README](README.md), "Inventory limitations"). Record this gap explicitly before implementing any destructive/write endpoint for papers — do not invent one from this summary alone.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [24.01 — Implement queries against the authorized Project DB, with pagination and search by key, title, author,…](sub-issues/24-01.md)
- [24.02 — Filter by Pending (default), Included, or Excluded decisions and return pagination metadata](sub-issues/24-02.md)
- [24.03 — Expose only id/key/title/URL/Source/decision and allowed actions in the list](sub-issues/24-03.md)
- [24.04 — Create the Paper key | Title | URL | Source | Decision | Actions table](sub-issues/24-04.md)
- [24.05 — Show ordered authors, DOI, URL, Source, Strategy, Venue, year, and abstract in details](sub-issues/24-05.md)
- [24.06 — Expose Import/Edit/Delete only to Owner/Manager when their respective flows exist](sub-issues/24-06.md)
