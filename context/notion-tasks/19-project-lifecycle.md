# 19 — Implement Project Lifecycle, Project Cards, and Metadata Overview

## Source and usage

- Source: [Implement Project Lifecycle, Project Cards, and Metadata Overview](https://app.notion.com/p/Implement-Project-Lifecycle-Project-Cards-and-Metadata-Overview-3ad499360293801eb322ddc6aae3f36c).
- Notion ID: `3ad499360293801eb322ddc6aae3f36c`.
- Source domain: Project & Protocol Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties plus its comment thread, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Provide a modern project list based on cover-image cards, explicit project actions, controlled project lifecycle transitions, and project metadata management.
- **Functional Domain:** Project & Protocol Management
- **Behavior:** Legacy ReLiS lists projects, lets users select a project, exposes view/detail/edit/delete actions, and supports publish/reopen behavior. The new ReLiS replaces hover-only actions with explicit card actions, replaces destructive deletion with archival, and keeps lifecycle transitions server-authorized and audited.
- **Users and permissions:** Any authenticated user can start a project-creation draft. A project record is created only after final creation confirmation. All active project members can see the project card and use Open workspace. Only the project creator and Application Administrators can access: Project details; Edit project; Publish / Unpublish project; Archive / Restore project. Project creator authority is checked only by the backend through `currentUserId === project.createdByUserId`.
- **UI behavior:**
  - Provide the global Projects page at `/projects`. Display: page title "Projects"; a short description; a **Create a new review** button; a search field; status filtering; sorting controls; a card-view and table-view switcher.
  - Create a new review opens the Project Creation Wizard.
  - Provide these tabs: **My projects** (projects where the current user is the Project Owner, plus the user's unfinished creation drafts); **My participating projects** (projects where the current user has an active role other than Project Owner); **Public projects** (published projects the current user can access without membership) — do not duplicate projects already shown in the other two tabs.
  - Apply search, filtering, sorting, and view selection within the currently selected tab.
  - Search projects by project title or Project Owner name. Filter by displayed status: `Draft`, `Provisioning`, `Active`, `Published`, `Archived`. Allow sorting by creation date or last modified date, ascending/descending.
  - A provisioning failure must return a safe failure state, displayed as `Provisioning failed` rather than exposing technical details.
  - **Card view** — each project card displays: project title; project short name; Project Owner name; current user's project profile; project status; last modified date. For a creation draft: draft title (or "Untitled review"); current user as Project Owner; `Draft` status; last modified date; a Resume draft action. For a provisioned project: a visible Open workspace action when the lifecycle state permits access. Lifecycle/metadata actions available only to authorised users: Project Owner (details, edit, publish, unpublish, archive, restore); Application Administrator (audited global lifecycle access).
  - **Table view** — same projects/actions in a table with columns `Project | Project Owner | My Role | Status | Last modified | Actions`; the Project column shows title and short name. Table and card views must show the same filtered/sorted result set.
  - **Metadata and lifecycle behavior** — Edit project opens the dedicated Project Details page; only the Project Owner can edit project metadata (title, description, research questions; project short name is read-only). Archive is reversible through an explicit restore action. Published projects are read-only for non-members accessing them through Public projects. Archived projects reject normal project writes. Lifecycle display states: `Draft`, `Provisioning`, `Active`, `Published`, `Archived`.
- **API behavior:** Provide operations to: list projects for the current tab; search by title/owner; filter by status; sort by creation/modified date; retrieve project details; update project metadata; publish/unpublish; archive/restore. The project-list response returns: title and short name; Project Owner display name; current user profile; lifecycle status; creation/last-modified date; authorised actions. Return creation drafts only in My projects — a draft is not a provisioned project and must not trigger Project DB access. Do not return project records the user cannot access, except published projects eligible for Public projects. Enforce Project Owner-only metadata editing. Never accept the Project Owner identity from the client and never allow it to change after project creation. Return safe provisioning/lifecycle states without exposing Project DB credentials or infrastructure details.
- **Data model:**
  - Control DB — project title; unique project short name; description; ordered research questions; protected Project Owner identity; lifecycle and publication state; creation and last-modified timestamps; archive metadata.
  - Project DB — accessed only after the project has been successfully provisioned and is available.
- **Acceptance criteria:**
  - [ ] Users see only projects for which they have active membership or global access.
  - [ ] Project lifecycle transitions follow the defined state machine and reject invalid transitions.
  - [ ] An archived project is read-only in the UI and API.
  - [ ] The overview accurately shows provisioning and enabled-module state.
  - [ ] Every lifecycle mutation has an audit event.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty
- **Comments:** Sokhna Khady MBAYE (2026-08-18): "Retravailler ce ticket en spécifiant de faire des tabs (voir photo)" — i.e. this ticket was flagged for rework to specify a tabbed layout (referencing a photo attachment not visible from this fetch). The current UI behavior text above already describes My projects / My participating projects / Public projects tabs, which may be the result of that requested rework, but re-check the live Notion page and its attached image for any visual/tab detail not captured by this text export before finalizing the UI.

## Objective and summarized requirements

Implement project lists, metadata, and lifecycle with server-verified creator authority.

## Proposed prerequisites

- [15 — Implement Project Context, Roles, and Membership](15-membership.md)
- [18 — Build the Native Project Protocol Wizard](18-protocol-wizard.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Create `/projects` with My projects (including drafts), My participating projects, and Public projects without duplicates.
2. Implement title/owner search, Draft/Provisioning/Active/Published/Archived filters, and creation/modification sorting; share the dataset between cards and table.
3. Return title, short name, owner, user profile, status, dates, and allowed actions; drafts have Resume and do not query the Project DB.
4. Implement details and editing for title, description, and research questions; short name and creator identity are immutable.
5. Implement publish/unpublish and archive/restore with valid transitions, confirmation, and audit; archived projects block writes.
6. Return safe provisioning failures and public access only according to publication policy.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the extensive UI behavior/API behavior fields, plus the open rework request noted in Comments.

- [ ] `/projects` renders My projects (owner + drafts), My participating projects, and Public projects with zero duplicate entries across tabs.
- [ ] Card view and table view read from the same filtered/sorted dataset — switching views never changes which projects appear.
- [ ] Search (title/owner), status filter (`Draft`/`Provisioning`/`Active`/`Published`/`Archived`), and sort (created/modified, asc/desc) all apply within the currently selected tab.
- [ ] A creation draft never triggers a Project DB query — its card/row shows a "Resume draft" action and `Draft` status derived purely from the Control DB.
- [ ] `currentUserId === project.createdByUserId` is checked only server-side; the client never supplies or can override the owner identity, and this check gates Project details/Edit/Publish/Unpublish/Archive/Restore.
- [ ] Editing project metadata (title, description, research questions) is possible only for the Project Owner; project short name is always read-only after creation.
- [ ] Archive is reversible via an explicit Restore action; an archived project rejects normal writes in both UI (disabled controls) and API (rejected mutation) — not only one of the two.
- [ ] Publishing makes a project read-only for non-members who access it via Public projects — they cannot mutate anything through that access path.
- [ ] A provisioning failure is surfaced as the generic `Provisioning failed` state — no PostgreSQL/infrastructure detail leaks into the list or details view.
- [ ] Every lifecycle mutation (publish, unpublish, archive, restore) writes an audit event.
- [ ] Resolve or explicitly flag the still-open rework request from the 2026-08-18 comment (tabbed layout) — re-check the live Notion page/attached photo before finalizing the tab UI, since this file's text export cannot capture the referenced image.
- [ ] Automated tests cover: tab de-duplication; owner-only edit rejection for a non-owner; archived-project write rejection; and an audit event created for each lifecycle transition.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

The source alternates creator plus Application Administrator access with owner-only editing; membership-only visibility criteria conflict with Public projects. Any user may create drafts here, while the wizard ([18](18-protocol-wizard.md)) restricts creation. Define policy before enabling public access or privileges — see [pending decisions](PENDING-DECISIONS.md) item 6. A 2026-08-18 Notion comment also requested a tabs-based rework referencing an attached photo not captured by this text export; verify the live page before finalizing UI details.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [19.01 — Create /projects with My projects (including drafts), My participating projects, and Public projects…](sub-issues/19-01.md)
- [19.02 — Implement title/owner search, Draft/Provisioning/Active/Published/Archived filters, and…](sub-issues/19-02.md)
- [19.03 — Return title, short name, owner, user profile, status, dates, and allowed actions](sub-issues/19-03.md)
- [19.04 — Implement details and editing for title, description, and research questions](sub-issues/19-04.md)
- [19.05 — Implement publish/unpublish and archive/restore with valid transitions, confirmation, and audit](sub-issues/19-05.md)
- [19.06 — Return safe provisioning failures and public access only according to publication policy](sub-issues/19-06.md)
