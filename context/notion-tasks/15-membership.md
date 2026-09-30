# 15 — Implement Project Context, Roles, and Membership

## Source and usage

- Source: [Implement Project Context, Roles, and Membership](https://app.notion.com/p/Implement-Project-Context-Roles-and-Membership-3ad49936029380d28180d33bc4573a5a).
- Notion ID: `3ad49936029380d28180d33bc4573a5a`.
- Source domain: Project & Protocol Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Make URL-based project context and project membership the authorization boundary for every project-scoped feature. Implement the legacy role model using Project Admin, Project Manager, Reviewer, Validator, and Guest, then replace the legacy global-user dropdown with exact account lookup and secure email invitations.
- **Functional Domain:** Project & Protocol Management
- **Behavior:** Legacy ReLiS assigns a creator an admin role, then lets project managers and Project Admin add users with one of four visible project roles: Project Manager, Reviewer, Validator, or Guest. Its user-selection UI exposes a dropdown containing all application users. NeoReLiS preserves the functional role model but uses canonical role values, exact lookup, invitation-based onboarding, and auditable project membership.
- **Users and permissions:**
  - The creator becomes Project Owner when project creation succeeds.
  - A Project Admin and Project Manager can add, invite, change the role of, and remove users only within their own project. Note: Project Manager cannot remove a Project Admin.
  - A Project Owner/Manager can assign: Project Manager, Reviewer, Validator, or Guest.
  - An Application Administrator has audited authority over every application user, invitation, project, and membership.
  - Reviewer, Validator, and Guest roles cannot manage project members or invitations.
  - A Project Manager has management, review, and validation capabilities; Reviewer and Validator permissions follow the project permission matrix; Guest is read-only access to all public project.
  - There is exactly one project role per active membership.
- **UI behavior:**
  - Provide a Project settings > Team page containing active members and outstanding invitations.
  - Show each entry's username, project role, status, added by, added date.
  - Do not show a searchable directory, dropdown, autocomplete list, pagination, or bulk export of application users.
  - Provide an Add team member form: select the intended project role; enter either an exact username or an exact email address.
  - When an exact username matches an existing account, show a confirmation card with its username and registered email, then allow the Project Manager to add that user to the project with the selected role.
  - When an exact email matches an existing account, show the matching username, then allow the Project Manager to add that user with the selected role.
  - When the username does not match an account, require an email address and keep the selected role.
  - The Invite user action performs a final email lookup before sending an invitation. If that email already belongs to an account, do not send an invitation; show the matching-account flow instead. If no account exists, create an invitation and show it with status `Invited`.
  - The invitation link expires after the invitation-validity period configured by the Application Administrator — by default it expires after 7 days.
  - Project Admin & Managers can resend or cancel an outstanding invitation, change a member's role, or remove a member.
  - Role changes and removal must warn about active work assignments. They must never erase prior decisions, assessments, extracted data, or audit history.
- **API behavior:**
  - Project-context middleware validates the project URL, project state, active membership, and role before executing any project-scoped use case. Application Administrators have audited global access.
  - Provide authorized use cases to: list project members and invitations; perform an exact username or email lookup; add an existing account to the current project; create, resend, cancel, and accept an invitation; change a project role; remove or restore a membership.
  - Account lookup must: require Project Admin & Manager or Application Administrator profile; accept one exact, normalized username or email value; return only the minimum confirmation information needed for the requested flow; never return a list of users or search suggestions.
  - Adding an existing account creates one active project membership after checking: the account exists; the account is not already an active member; the requested role is valid; the action does not violate the last-Project-Manager guard.
  - Creating an invitation must: use the normalized email address and selected role; create a `ProjectInvitation` record, not a membership for a non-existent user; generate a single-use, securely random invitation token; store only a hash of the token; expire after 7 days by default if no invitation-validity period was configured by the Application Administrator; invalidate any previous active invitation for the same project and email when resent; send the invitation email asynchronously.
  - Accepting an invitation must: validate the token, expiration, invitation status, and email ownership; require registration or sign-in with the invited email address; create one active membership with the invitation's assigned role; mark the invitation `Accepted`; never allow the invitation email or role to be silently changed.
- **Data model:**
  - Control DB — Global User identity: immutable ID, unique normalized username, unique normalized email, email-verification state, account status, and timestamps.
  - Control DB — Project Membership: project ID, user ID, role (`PROJECT_ADMIN`, `PROJECT_MANAGER`, `REVIEWER`, `VALIDATOR`, `GUEST`), status (`ACTIVE`, `REMOVED`), actor, timestamps, and removal reason.
  - Control DB — Project Invitation: project ID, normalized email, proposed role, status (`INVITED`, `ACCEPTED`, `EXPIRED`, `CANCELLED`), token hash, expiration time, inviter, accepted user ID, sent/accepted timestamps.
  - Membership and invitation audit events.
  - Enforce one active membership per `(projectId, userId)`.
  - Enforce at most one active invitation per `(projectId, normalizedEmail)`.
  - Project DB: contains no user identities, passwords, invitations, roles, or membership copies.
- **Acceptance criteria:**
  - [ ] A project-scoped API request is authorized from the URL project ID and active membership.
  - [ ] Administrators and Project Admins can manage membership only within their permitted scope.
  - [ ] Duplicate active membership is prevented.
  - [ ] Guests have read-only access and cannot access mutation endpoints.
  - [ ] Removing the last Project Admin is rejected.
- **Assigned to:** Empty
- **Severity:** Minor
- **Blocked by:** Empty
- **Jobs behavior:**
  - Send invitation and resend emails through the existing transactional email job mechanism.
  - Expire invitations after 48 hours through expiry checks and scheduled cleanup; expired invitations cannot be accepted.
  - Failed email delivery must be visible to authorized managers without exposing SMTP/provider internals.
  - Retries must not create duplicate invitations or send multiple valid tokens unintentionally.
- **Audit:**
  - exact account lookup; member added; invitation created, sent, resent, cancelled, expired, and accepted; role changed; member removed or restored; rejected duplicate membership, invalid invitation, and last-Project-Manager actions.
  - Removing a membership revokes access but never deletes the global account or historic project work. Restoring a removed member is an explicit, audited action.

## Objective and summarized requirements

Use the URL project and current membership as the authorization boundary, with exact account lookup and secure invitations.

## Proposed prerequisites

- [03 — Create control Prisma schema](03-control-prisma.md)
- [05 — Implement API app shell](05-api-shell.md)
- [06 — Complete job queue and progress platform](06-job-platform.md)
- [09 — Implement Secure Login, Logout, and Session Management](09-login-sessions.md)
- [10 — Implement Registration and Email Verification](10-registration.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Define middleware that verifies the URL `projectId`, state, membership, and role; allow audited global support access.
2. Implement Team, exact normalized username/email lookup, and adding existing accounts; do not offer a global user directory or autocomplete.
3. Implement invitations with a fixed role, random token stored as a hash, expiry, and asynchronous email; check again for an existing account before inviting.
4. Accept an invitation only with verified ownership of its email; consume it once and create one active membership.
5. Prevent duplicate active memberships and invitations; resending invalidates the previous invitation.
6. Implement role changes, removal, and restoration while preserving historical work; show assignment impact and audit actions, including lookup.
7. Keep identities, credentials, invitations, memberships, and roles only in the Control DB.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/Data model/Jobs behavior/Audit fields, which carry unusually rich detail for this card.

- [ ] Every project-scoped route resolves authorization from the URL `projectId` + the caller's active membership + role — an ID belonging to a different project can never be used to access data across the boundary.
- [ ] `Team` shows active members and outstanding invitations (username, role, status, added-by, added-date) but never a searchable/autocomplete directory of all application users.
- [ ] Account lookup accepts only one exact, normalized username or email, returns the minimum confirmation info, and never returns a list/search suggestions.
- [ ] Inviting re-checks for an existing account at send time: if the email now matches an account, the invite is not sent and the matching-account flow is shown instead.
- [ ] Invitation tokens are single-use, securely random, stored only as a hash, and resending invalidates any previous active invitation for the same `(project, email)` pair.
- [ ] Accepting an invitation requires verified ownership of the invited email, creates exactly one active membership with the invitation's role, and marks the invitation `Accepted` — the email or role can never be silently changed during acceptance.
- [ ] At most one active membership exists per `(projectId, userId)`, and at most one active invitation per `(projectId, normalizedEmail)` — enforced at the data layer, not just in the UI.
- [ ] Guests have read-only access; every mutation endpoint (invite, role change, remove, resend, cancel) rejects Reviewer, Validator, and Guest callers.
- [ ] Removing the last Project Admin (or, per the role-cycle decision recorded in [pending decisions](PENDING-DECISIONS.md), the equivalent protected owner role) is rejected; a Project Manager cannot remove a Project Admin.
- [ ] Removing a membership revokes access without deleting the global user account or any historical project work/attribution; restoring a removed member is an explicit, audited action.
- [ ] Invitation and resend emails are sent asynchronously through the shared job platform ([06](06-job-platform.md)); failed delivery is visible to authorized managers without exposing SMTP/provider internals, and retries do not create duplicate invitations or multiple valid tokens.
- [ ] The resolved invitation-expiry period (7 days per UI/API vs. 48 hours per Jobs behavior — see Gaps) is applied consistently across token generation, UI display, and scheduled cleanup.
- [ ] Every listed action (lookup, member added, invitation lifecycle events, role changed, member removed/restored, and every rejected duplicate/invalid/last-Project-Manager attempt) writes an audit event.
- [ ] All of the above data (identities, credentials, invitations, memberships, roles) lives only in the Control DB — never duplicated into a Project DB.
- [ ] Automated tests cover: cross-project access rejection; duplicate membership/invitation rejection; full invite→accept flow; last-Project-Admin removal rejection; and a Guest's 403 on every mutation endpoint.
- [ ] Report which commands were run and their actual results, and record which invitation-expiry value (7 days vs. 48 hours) and which role model (`PROJECT_ADMIN` vs. protected `PROJECT_OWNER`) were actually chosen, per the decisions required below.

## Gaps and pending decisions

Conflicts block the final contract: `PROJECT_ADMIN` versus protected `PROJECT_OWNER`; last Project Manager versus last Project Admin guard; invitations last 7 days in UI/API versus 48 hours in Jobs behavior. Project Managers cannot remove Project Admins. See [PENDING-DECISIONS.md](PENDING-DECISIONS.md).

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [15.01 — Define middleware that verifies the URL projectId, state, membership, and role](sub-issues/15-01.md)
- [15.02 — Implement Team, exact normalized username/email lookup, and adding existing accounts](sub-issues/15-02.md)
- [15.03 — Implement invitations with a fixed role, random token stored as a hash, expiry, and asynchronous email](sub-issues/15-03.md)
- [15.04 — Accept an invitation only with verified ownership of its email](sub-issues/15-04.md)
- [15.05 — Prevent duplicate active memberships and invitations](sub-issues/15-05.md)
- [15.06 — Implement role changes, removal, and restoration while preserving historical work](sub-issues/15-06.md)
- [15.07 — Keep identities, credentials, invitations, memberships, and roles only in the Control DB](sub-issues/15-07.md)
