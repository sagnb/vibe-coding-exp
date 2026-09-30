# 26 — Implement Venue Directory

## Source and usage

- Source: [Implement Venue Directory](https://app.notion.com/p/Implement-Venue-Directory-3c6499360293809087fff284e927dd22).
- Notion ID: `3c6499360293809087fff284e927dd22`.
- Source domain: Paper Library & Import.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Provide a project-scoped Venue page for viewing and managing publication venues used by Papers. A venue stores its full name and optional publication year. It may be added or updated manually by authorised project managers.
- **Functional Domain:** Paper Library & Import
- **Behavior:** Empty
- **Users and permissions:**
  - All authorised project members can view the Venue page.
  - Only the Project Owner, Project Managers, and authorised Application Administrators can enter Edit mode, add venues, edit venues, or delete venues.
  - All data is scoped to the current project.
- **UI behavior:**
  - Within Paper library > Venues, display a searchable, paginated table: Full venue name | Publication year | Actions. Full venue name is required; Publication year is optional. The table supports sorting by venue name and publication year.
  - **Edit mode** — shown only to authorised users; existing venue fields become editable inline. Provide "Add venue" (editable row: full venue name required, publication year optional). Provide Delete with a confirmation prompt. Save changes directly to the Project DB when the user saves a row.
  - **Integrity rules** — editing a venue updates the shared venue record displayed by every linked Paper. A venue linked to one or more Papers cannot be deleted (display a clear explanation instead).
- **API behavior:**
  - Use Hono as the authenticated API layer. Enforce current project context and role permissions for every operation.
  - Provide use cases to: list venues; create venues; update venues; delete unreferenced venues. Use normal Prisma CRUD against the Project DB fixed schema. Reject deletion of a venue referenced by Papers with a safe, human-readable error. Record creation and update audit information.
- **Data model:**
  - Venue — full name; optional publication year; created/updated audit fields.
  - Paper — optional Venue reference.
- **Acceptance criteria:**
  - [ ] The Venues page displays full venue name and publication year for the current project only.
  - [ ] The table is searchable, paginated, and sortable.
  - [ ] Authorised users can add and edit venues in Edit mode.
  - [ ] A venue linked to a Paper cannot be deleted.
  - [ ] Reviewers, Validators, and Guests cannot modify venue data.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Manage project-scoped venues with a full name and optional year.

## Proposed prerequisites

- [24 — Implement Project Paper List](24-paper-list.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Reuse Venue and the optional Paper reference in the fixed schema.
2. Implement list/create/update/delete for unreferenced venues, verifying project and role.
3. Create the Full venue name | Publication year | Actions table with search, pagination, and sorting.
4. Allow Edit mode and new rows for Owner/Manager/Admin; validate the required name and submitted fields.
5. Block deletion when a Paper references the venue; audit creation/updates and preserve shared references.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the UI behavior/Data model fields.

- [ ] The Venues table shows exactly Full venue name | Publication year | Actions, scoped strictly to the current project — a venue belonging to another project can never appear here.
- [ ] Search, pagination, and sorting (by venue name and by publication year) all work and compose correctly.
- [ ] Edit mode and every add/edit/delete action are visible and reachable only for the Project Owner, Project Managers, and authorised Application Administrators — Reviewers, Validators, and Guests can view the page but see no mutation controls, and their direct API calls are rejected server-side too.
- [ ] Editing a venue's name or year updates the single shared `Venue` record — every Paper referencing it reflects the change immediately.
- [ ] Deleting a venue referenced by one or more Papers is rejected with a clear, human-readable explanation — not a raw database constraint error.
- [ ] Venue name is required on create/edit; publication year remains optional.
- [ ] Every create/update writes creation/update audit information (`createdAt`/`createdBy`, `updatedAt`/`updatedBy` per the Data model).
- [ ] Automated tests cover: search/sort/pagination correctness; a blocked deletion of a referenced venue; a successful edit reflected on a linked Paper; and a Reviewer/Validator/Guest's 403 on every mutation endpoint.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

No specific contradiction is highlighted in this summary. Cross-cutting decisions in the pending-decisions register still apply. Empty source fields (Behavior) do not authorize expanding scope.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [26.01 — Reuse Venue and the optional Paper reference in the fixed schema](sub-issues/26-01.md)
- [26.02 — Implement list/create/update/delete for unreferenced venues, verifying project and role](sub-issues/26-02.md)
- [26.03 — Create the Full venue name | Publication year | Actions table with search, pagination, and sorting](sub-issues/26-03.md)
- [26.04 — Allow Edit mode and new rows for Owner/Manager/Admin](sub-issues/26-04.md)
- [26.05 — Block deletion when a Paper references the venue](sub-issues/26-05.md)
