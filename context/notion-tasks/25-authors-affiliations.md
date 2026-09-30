# 25 — Implement Authors and Affiliations Directory

## Source and usage

- Source: [Implement Authors and Affiliations Directory](https://app.notion.com/p/Implement-Authors-and-Affiliations-Directory-3c64993602938073a428fd6b23b5fb48).
- Notion ID: `3c64993602938073a428fd6b23b5fb48`.
- Source domain: Paper Library & Import.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Provide a project-scoped Authors page with directories for all authors, first authors, and affiliations. Paper counts are derived from active Paper and PaperAuthor records; they are never manually editable. Authors and affiliations may be added, edited, and deleted.
- **Functional Domain:** Paper Library & Import
- **Behavior:** Empty
- **Users and permissions:**
  - All authorised project members can view the Authors page.
  - Only the Project Owner, Project Managers, and Application Administrators can enter Edit mode, add records, edit records, or delete records.
  - All data is scoped to the current project.
- **UI behavior:**
  - Within Paper library > Authors, provide two tabs: 1. Authors; 2. Affiliations.
  - **Authors** — segmented filter "All authors | First authors" above the table; the selected filter updates rows and the count column.
    - All authors: searchable, paginated table Author | Affiliation | Papers | Actions. Author: display name. Affiliation: linked affiliation when available, otherwise `—`. Papers: number of active project Papers where the author appears in the ordered author list. Includes manually added authors, including authors with 0 Papers. Counts are read-only.
    - First authors: searchable, paginated table Author | Affiliation | First-author papers | Actions. First-author papers: number of active project Papers where the author appears in first position. Includes only authors whose count is greater than 0. Counts are read-only.
  - **Affiliations** — searchable, paginated table Institute | Description | Actions. Institute is required; Description is optional.
  - **Edit mode** — shown only to authorised users; editable cells become structured inputs; save changes directly to the Project DB; counts remain read-only. Provide "Add author" in the Authors tab: new editable row with author name (required) and optional affiliation; when created while the First authors filter is active, switch to All authors so the new author is visible. Provide "Add affiliation" in the Affiliations tab: new editable row with institute (required) and optional description. Provide Delete actions with a confirmation prompt.
  - **Integrity rules** — editing an author updates the shared author record displayed by every linked Paper. An author linked to one or more Papers cannot be deleted (display a clear explanation). An affiliation linked to one or more Authors cannot be deleted (display a clear explanation). This ticket does not edit Paper author order or add/remove author-to-Paper links — those actions belong to Paper editing (a task not present among the 42 consulted pages; see Gaps).
- **API behavior:**
  - Use Hono as the authenticated API layer. Enforce current project context and role permissions for every operation. Allow all authorised project members to read Authors and Affiliations. Allow only the Project Owner, Project Managers, and authorised Application Administrators to create, update, or delete Authors and Affiliations.
  - Provide use cases to: list Authors with `scope=all` for all active project authors, `scope=first` for authors who are first author on at least one active project Paper, plus search/pagination/sorting; list Affiliations with search/pagination/sorting; create/update/delete an Author; create/update/delete an Affiliation.
  - For Author lists: with `scope=all`, return author name, optional affiliation, and derived `paperCount`; with `scope=first`, return author name, optional affiliation, and derived `firstAuthorPaperCount`. Derive `paperCount` from active `PaperAuthor` links to active project Papers. Derive `firstAuthorPaperCount` from active `PaperAuthor` links at author position 1. Do not persist either count as an editable Author field. Manual Authors may be created without `PaperAuthor` links and must appear in `scope=all`.
  - For mutations: validate submitted fields with Zod; use normal Prisma CRUD against the Project DB fixed schema; update the shared Author or Affiliation record directly; reject deletion of an Author linked to one or more Papers; reject deletion of an Affiliation linked to one or more Authors; return the updated row or structured field-level validation errors; write an audit event for every create, update, and delete action.
- **Data model:**
  - Author — id; displayName (required); affiliationId (optional reference to Affiliation); createdAt, createdBy; updatedAt, updatedBy.
  - Affiliation — id; institute (required); description (optional); createdAt, createdBy; updatedAt, updatedBy.
  - PaperAuthor — paperId; authorId; position (positive integer representing the author's order on the Paper).
  - Paper — linked to Authors only through PaperAuthor.
  - Derived values: `paperCount` from active PaperAuthor links to active project Papers; `firstAuthorPaperCount` from active PaperAuthor links where `position = 1`. Do not store either count as a mutable database column.
  - Integrity rules: an Author with PaperAuthor links cannot be deleted; an Affiliation referenced by one or more Authors cannot be deleted; Authors and Affiliations exist only in the current Project DB.
- **Acceptance criteria:** Empty
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Create author and affiliation directories with counts derived from active Papers.

## Proposed prerequisites

- [24 — Implement Project Paper List](24-paper-list.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Model/reuse Author, Affiliation, and ordered PaperAuthor records in the Project DB.
2. Implement `scope=all` and `scope=first` lists; derive counts from active links to active Papers and author position 1.
3. Create Authors with All authors/First authors, search/pagination/sorting, and Affiliation; manually created authors with no Papers appear in All.
4. Create Affiliations with required Institute and optional Description.
5. Allow inline editing/addition/deletion only for Owner/Manager/Admin, with Zod, auditing, and confirmation.
6. Block deletion of authors linked to Papers and affiliations linked to authors; do not change Paper authorship/order here.

## Definition of Done — completion checklist

Notion's Acceptance criteria property is empty; the checklist below combines that fact with concrete points grounded in the Data model/API behavior fields above, which carry unusually precise derivation rules.

- [ ] `paperCount` and `firstAuthorPaperCount` are computed on read from active `PaperAuthor` links (the latter filtered to `position = 1`) — neither is ever stored as a mutable, directly editable field on `Author`.
- [ ] `scope=all` includes every active author, including manually added ones with zero linked Papers; `scope=first` includes only authors whose `firstAuthorPaperCount` is greater than zero.
- [ ] Adding a new author while the "First authors" filter is active switches the view to "All authors" so the newly created (zero-count) author is actually visible.
- [ ] Only the Project Owner, Project Managers, and authorised Application Administrators can enter Edit mode or call any create/update/delete endpoint; all other authorised members have read-only access.
- [ ] Editing an author's name or affiliation updates the single shared `Author` record — every Paper that links to that author reflects the change, there is no per-paper copy.
- [ ] Deleting an Author with any active `PaperAuthor` link is rejected with a clear explanation; deleting an Affiliation referenced by any Author is rejected with a clear explanation.
- [ ] This feature never modifies `PaperAuthor.position` or adds/removes author-to-Paper links — those remain explicitly out of scope here (see Gaps).
- [ ] Every create/update/delete on Author or Affiliation is validated with Zod, persisted via normal Prisma CRUD against the already-fixed Project DB schema, and writes an audit event.
- [ ] Automated tests cover: `scope=all` vs `scope=first` filtering and counts; a blocked deletion of a referenced author/affiliation; a successful edit reflected across multiple linked papers; and a non-authorised role's 403 on every mutation endpoint.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

Acceptance criteria are empty in the source; the checklist is proposed from UI/API requirements. Editorial questions about authors without Papers and affiliations are explicitly resolved by the API field, which includes these cases. Paper author-order editing and author-to-Paper linking belong to a "Paper editing" capability that has no complete ticket among the 42 consulted Notion pages (see the [notion-tasks README](README.md), "Inventory limitations") — do not implement it here.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [25.01 — Model/reuse Author, Affiliation, and ordered PaperAuthor records in the Project DB](sub-issues/25-01.md)
- [25.02 — Implement scope=all and scope=first lists](sub-issues/25-02.md)
- [25.03 — Create Authors with All authors/First authors, search/pagination/sorting, and Affiliation](sub-issues/25-03.md)
- [25.04 — Create Affiliations with required Institute and optional Description](sub-issues/25-04.md)
- [25.05 — Allow inline editing/addition/deletion only for Owner/Manager/Admin, with Zod, auditing, and confirmation](sub-issues/25-05.md)
- [25.06 — Block deletion of authors linked to Papers and affiliations linked to authors](sub-issues/25-06.md)
