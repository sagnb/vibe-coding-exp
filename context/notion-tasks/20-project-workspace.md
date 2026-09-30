# 20 — Implement Project Workspace Navigation and Home Dashboard

## Source and usage

- Source: [Implement Project Workspace Navigation and Home Dashboard](https://app.notion.com/p/Implement-Project-Workspace-Navigation-and-Home-Dashboard-3ad49936029380119f60ce93f1f5d75f).
- Notion ID: `3ad49936029380119f60ce93f1f5d75f`.
- Source domain: Project & Protocol Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Implement the persistent project workspace shell, role-aware navigation, and Project Home dashboard. Every active member can enter the workspace; available navigation and actions adapt to role, lifecycle state, and enabled protocol modules.
- **Functional Domain:** Project & Protocol Management
- **Behavior:** Legacy ReLiS switches to a project-specific working perspective, shows a project header and phase table, and presents screening, QA, classification, participants, and project actions. NeoReLiS keeps the project workspace concept while replacing session-based project selection with URL-based context and explicit authorization.
- **Users and permissions:**
  - Every active project member can use Open workspace.
  - Project Managers receive operational workspace capabilities beyond those of Reviewers, Validators, and Guests: Team management, invitations, paper import, and operational phase controls.
  - Reviewers and Validators access only the workflow actions and assigned work permitted by their role.
  - Guests receive read-only workspace access.
  - Application Administrators have audited global access.
  - Creator-only metadata and lifecycle actions remain outside this ticket and are enforced by [19 — Project Lifecycle](19-project-lifecycle.md).
- **UI behavior:**
  - One persistent authenticated application shell across global and project pages. Within `/projects/:projectId`, the top header contains: ReLiS branding; a hamburger button to collapse/expand the left navigation; a project switcher showing the current project short name; language selection; the current user avatar and menu.
  - The project switcher lets the user navigate to another accessible project workspace. Leaving the project workspace for Dashboard or Projects keeps the same shell, replacing project-specific navigation with global navigation.
  - **Left navigation** — two levels: Workspace (Dashboard, Projects) and Current project (Project home, Paper library [Papers, Import papers, Authors, Venues], Screening, Quality assessment, Data extraction, Reporting, Team, Administration [visible to application admin]).
  - Paper library is a collapsible navigation group. Show project-specific navigation only within an explicit project route. Show a module entry only when the active project protocol enables the corresponding module and the current user is authorised to access it. Import papers and Team are available to the Project Owner and Project Managers only. Administration shows only entries the current user is authorised to access. Hiding a navigation item never replaces backend authorisation.
  - **Project Home** — breadcrumb `Projects / [project short name] / Project home`; displays project title, description, and current state (`Active`/`Archived`/`Published`). A Project Owner card shows profile picture (or avatar), first/last name, email, and the "Project Owner" label — this is protected backend-managed state, not assignable/editable by Project Managers. Show "Edit project" only to the Project Owner, opening the dedicated Project Details page.
  - **Review workflow** — a table `Phase | Status | My completion | Overall completion | Action` with one row per configured screening phase (labelled "Screening: [phase title]"), one Quality Assessment row when configured, and one Classification row when enabled; display `—` for Status/My completion/Overall completion in this ticket; display an Open phase button only for the Project Owner and Project Managers.
  - **Participants** — active project members via the reusable compact person-card component, each showing their assigned role; exclude the Project Owner (already shown in the Owner card). Show "View team" (linking to the Team page) only to the Project Owner and Project Managers. Outstanding invitations belong to the Team page, not Project Home.
- **API behavior:** Provide an authenticated project-shell endpoint returning: project identity (ID, short name, title, description, lifecycle state); current user's project access profile (assigned role, `isProjectOwner`); whether the current user is an Application Administrator; enabled project modules; authorised navigation entries; authorised Project Home actions (`canEditProject`, `canViewTeam`, `canOpenPhase`); Project Owner summary (profile picture reference, name, email); configured workflow display entries (screening phase titles; whether QA is configured; whether Classification is enabled). The Project Details API enforces Project-Owner-only metadata edits. Provisioning or failed projects return a safe workspace state and must not query an unavailable Project DB.
- **Data model:**
  - Control DB: project lifecycle, creator identity, current user membership/role, cover image reference.
  - Project DB: active protocol metadata, enabled modules, screening phase state, QA/classification state, assignments, completion data, and participant-related project data.
- **Acceptance criteria:**
  - Every active member can open the project workspace from a project card.
  - The workspace header and collapsible navigation remain visible across project routes.
  - Navigation adapts to membership role, lifecycle state, and enabled modules.
  - Import Papers and Team are inaccessible to Reviewers, Validators, and Guests.
  - Project Home displays the description, creator card, phase table, and participants list.
  - Phase actions are restricted by role and lifecycle state.
  - Completion metrics do not mix different workflow assignment types.
  - Disabled modules are hidden from navigation and rejected by their APIs.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Create a persistent shell, navigation by role/module/state, and Project Home.

## Proposed prerequisites

- [15 — Implement Project Context, Roles, and Membership](15-membership.md)
- [17 — Implement project database resolver](17-project-resolver.md)
- [18 — Build the Native Project Protocol Wizard](18-protocol-wizard.md)
- [19 — Implement Project Lifecycle, Project Cards, and Metadata Overview](19-project-lifecycle.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Integrate the authenticated global/project shell with branding, collapsible navigation, URL-based switcher, language, and user menu.
2. Provide a shell endpoint with identity, lifecycle, membership, `isProjectOwner`, global authority, modules, actions, and owner summary.
3. Create global Dashboard/Projects navigation and project Home, Papers/Imports/Authors/Venues, Screening, QA, Data Extraction, Reporting, Team, and authorized administration.
4. Create Home with breadcrumb, title/description/state, owner card with avatar/name/email, and participants without duplicating the owner.
5. Display the configured Review workflow phases, QA, and Classification; this delivery uses `—` for status/completion and restricts Open phase as specified in the source.
6. Switching projects invalidates visual/data context and navigates by URL; unavailable projects must not query incomplete databases.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the very detailed UI behavior/API behavior fields.

- [ ] Every active member can reach the workspace from a project card, and the header/collapsible left navigation persists across every project route (not re-rendered/reset per page).
- [ ] The shell endpoint returns everything the UI needs to render navigation/actions without extra round trips: project identity, current user's role and `isProjectOwner`, global-admin flag, enabled modules, authorised navigation entries, `canEditProject`/`canViewTeam`/`canOpenPhase`, Owner summary, and workflow display entries.
- [ ] A navigation item for a disabled module never renders, **and** its corresponding API route independently rejects direct calls — hiding is not the only enforcement.
- [ ] Import Papers and Team are absent from navigation for Reviewers, Validators, and Guests, and their APIs reject those roles even via direct URL/API call.
- [ ] Project Home shows: breadcrumb with the project short name; title/description/lifecycle state; the Owner card (picture/avatar, name, email, "Project Owner" label) which is never editable by a Project Manager; the Review workflow table with one row per configured screening phase plus QA/Classification rows as applicable, showing `—` for Status/My/Overall completion in this ticket (no premature metric computation); and the Participants list excluding the Owner.
- [ ] "Edit project" is visible only to the Project Owner; "Open phase" is visible only to the Project Owner and Project Managers; "View team" is visible only to the Project Owner and Project Managers.
- [ ] Switching projects via the project switcher fully invalidates the previous project's cached context (no stale data bleeding from project A into project B's view) and navigates via the URL, consistent with [17](17-project-resolver.md)'s resolver.
- [ ] A provisioning/failed project renders a safe workspace state and never triggers a query against its (unavailable) Project DB.
- [ ] Completion metrics, when eventually wired, must not mix different workflow assignment types (screening vs. QA vs. classification) into one combined number — this ticket itself only needs to avoid computing them prematurely (`—` placeholders).
- [ ] Automated tests cover: navigation visibility per role (Owner/Manager/Reviewer/Validator/Guest/Admin); a disabled module rejected via direct API call; Project Home rendering all required sections; and project-switch context invalidation.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

Acceptance criteria mention metrics, but the UI requires `—` placeholders in this task. Integrate metrics only with their own tickets ([32](32-screening-metrics.md), [34](34-qa-results.md), [36](36-classification-progress.md)). Administrator editing exceptions must be reconciled with lifecycle rules from [19](19-project-lifecycle.md).

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [20.01 — Integrate the authenticated global/project shell with branding, collapsible navigation, URL-based…](sub-issues/20-01.md)
- [20.02 — Provide a shell endpoint with identity, lifecycle, membership, isProjectOwner, global authority, modules,…](sub-issues/20-02.md)
- [20.03 — Create global Dashboard/Projects navigation and project Home, Papers/Imports/Authors/Venues, Screening,…](sub-issues/20-03.md)
- [20.04 — Create Home with breadcrumb, title/description/state, owner card with avatar/name/email, and participants…](sub-issues/20-04.md)
- [20.05 — Display the configured Review workflow phases, QA, and Classification](sub-issues/20-05.md)
- [20.06 — Switching projects invalidates visual/data context and navigates by URL](sub-issues/20-06.md)
