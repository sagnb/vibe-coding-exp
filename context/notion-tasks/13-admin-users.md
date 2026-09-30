# 13 — Implement Administrator User Directory and Account Lifecycle Management

## Source and usage

- Source: [Implement Administrator User Directory and Account Lifecycle Management](https://app.notion.com/p/Implement-Administrator-User-Directory-and-Account-Lifecycle-Management-3ac49936029380f2ae06dd6208248e09).
- Notion ID: `3ac49936029380f2ae06dd6208248e09`.
- Source domain: Authentication & User Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Provide an Application Administrator-only user directory for creating, searching, viewing, updating, assigning application authority, archiving/restoring accounts, and triggering a password reset. Historical review attribution must survive account archival.
- **Functional Domain:** Authentication & User Management
- **Behavior:** Legacy ReLiS supports add/edit/view/archive users, user groups, and password management. The new relis preserves administrative lifecycle management but avoids destructive user deletion and direct administrator password setting.
- **Users and permissions:**
  - Only Application Administrators can access the system-wide user/project directory and assign application authority.
  - Application Administrators cannot archive the final active Application Administrator account.
  - Archived users cannot log in or receive new assignments, but historical project data remains attributed to them.
- **UI behavior:**
  - Paginated, searchable user directory with account status, verification state, application authority, project-membership summary, and audit history link.
  - User detail page supports create/edit allowed fields, archive/restore, role changes, and force-reset action.
  - High-impact actions use confirmation dialogs explaining their consequences.
- **API behavior:**
  - Provide list-users, get-user, create-user, update-user-admin, assign-application-administrator, archive-user, restore-user, and force-password-reset? use cases.
  - Reject permanent deletion from normal administration flows.
  - Enforce last-active-administrator protection and prohibit editing through self-service endpoints.
- **Data model:** Control DB: users, global roles, active/archive state, timestamps, administrator action metadata, and account audit events. Project DB: no user deletion; project review records retain their original user identity/reference.
- **Acceptance criteria:**
  - [ ] Application Administrators can create, search, view, update, archive, and restore any application user with a full audit trail.
  - [ ] Archived users cannot log in or receive new project assignments, while historical data remains attributed.
  - [ ] The final active Application Administrator cannot be archived or demoted.
  - [ ] Application Administrator password reset invokes the secure reset flow and never reveals or directly sets a password.
  - [ ] Project Admins cannot access system-wide user-directory APIs or manage users outside their own project.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Create an administrator directory and account lifecycle management while preserving historical attribution.

## Proposed prerequisites

- [09 — Implement Secure Login, Logout, and Session Management](09-login-sessions.md)
- [11 — Implement Password Change, Recovery, and Administrator Force Reset](11-password-reset.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Implement list-users, get-user, create-user, update-user-admin, and global-authority assignment for Application Administrators only.
2. Protect the last active administrator from archival or authority removal.
3. Implement archive/restore without permanent deletion; prevent archived users from logging in or receiving new assignments.
4. Integrate force-password-reset with the existing secure flow.
5. Create a searchable, paginated directory, details, memberships, status, verification, history, and high-impact confirmations.
6. Audit actions and preserve historical IDs in project records.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the UI behavior/API behavior fields.

- [ ] Every endpoint in this feature (`list-users`, `get-user`, `create-user`, `update-user-admin`, `assign-application-administrator`, `archive-user`, `restore-user`, force-password-reset) is reachable only by Application Administrators — a Project Admin or lower role gets a clean 403, not a filtered result set.
- [ ] Archiving the last active Application Administrator is rejected server-side (not just hidden in the UI) with a clear error.
- [ ] Archived users cannot authenticate ([09](09-login-sessions.md)) and cannot be assigned to a new project; their historical `createdBy`/review attribution remains intact and readable (no deletion, no anonymization here).
- [ ] There is no permanent/hard delete path for users anywhere in this feature — only archive/restore.
- [ ] The administrator-triggered password reset reuses the secure reset flow from [11](11-password-reset.md); it never displays, emails to the admin, or lets the admin set a password directly.
- [ ] High-impact actions (archive, restore, authority change, force reset) require an explicit confirmation step in the UI that states the consequence (e.g., "this user will lose access immediately").
- [ ] The directory is paginated and searchable, and each user row/detail surfaces account status, verification state, application authority, a project-membership summary, and a link to that user's audit history.
- [ ] Every administrative action here (create/update/archive/restore/authority change/force reset) writes a Control DB audit event with actor, target, and outcome.
- [ ] Self-service endpoints ([12](12-profile.md)) cannot be used to change fields that only this admin surface should control (global authority, active/archive status, verification state) — cross-check against 12's allowlist.
- [ ] Automated tests cover: last-administrator archive rejection; archived user login rejection; force-reset never exposing a password; and a non-administrator's 403 on every listed endpoint.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

No specific contradiction is highlighted in this summary. Cross-cutting decisions in the pending-decisions register still apply. Empty source fields do not authorize expanding scope.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [13.01 — Implement list-users, get-user, create-user, update-user-admin, and global-authority assignment for…](sub-issues/13-01.md)
- [13.02 — Protect the last active administrator from archival or authority removal](sub-issues/13-02.md)
- [13.03 — Implement archive/restore without permanent deletion](sub-issues/13-03.md)
- [13.04 — Integrate force-password-reset with the existing secure flow](sub-issues/13-04.md)
- [13.05 — Create a searchable, paginated directory, details, memberships, status, verification, history, and…](sub-issues/13-05.md)
- [13.06 — Audit actions and preserve historical IDs in project records](sub-issues/13-06.md)
