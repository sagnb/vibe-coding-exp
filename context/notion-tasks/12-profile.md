# 12 — Implement Self-Service Profile and Account Preferences

## Source and usage

- Source: [Implement Self-Service Profile and Account Preferences](https://app.notion.com/p/Implement-Self-Service-Profile-and-Account-Preferences-3ac4993602938004b9d2dd0b4a13c8a0).
- Notion ID: `3ac4993602938004b9d2dd0b4a13c8a0`.
- Source domain: Authentication & User Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Implement Secure Login, Logout, and Session Management.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Give each user a self-service account page to view and update permitted profile details, profile picture, language preference, and password entry point—without allowing role or account-status changes.
- **Functional Domain:** Authentication & User Management
- **Behavior:** Legacy ReLiS lets users edit email, password, and profile picture. The new relis separates self-service profile data from privileged account state and uses the shared validated storage service for profile images. Note: profile changes do not alter project membership or permissions. The selected locale updates project screens through the localization resolver.
- **Users and permissions:** All authenticated active users can view and edit only their own permitted profile fields. They cannot change Application Administrator authority, project role, active/archive status, verification state, or another user's account.
- **UI behavior:**
  - Account page shows profile identity, email, language, picture, account verification/status summary, and password-security link.
  - Profile-picture upload validates file type and size, supports replacement/removal, and presents accessible failure states.
  - Changing email initiates a verified change flow rather than immediately replacing a verified address.
- **API behavior:**
  - Provide get-my-profile, update-my-profile, start-email-change, confirm-email-change, upload-profile-picture, and remove-profile-picture use cases.
  - Reuse the verification mail adapter for an email-address change.
  - Use storage abstraction; no local filesystem paths are exposed to the browser.
- **Data model:** Control DB: user display/profile fields, verified email, pending-email-change token metadata, preferred locale, and profile-image storage reference. File storage: validated profile image object and metadata.
- **Acceptance criteria:** Empty
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Implement Secure Login, Logout, and Session Management
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Allow users to manage their profile details, picture, language, and verified email changes.

## Proposed prerequisites

- [07 — Implement upload/storage service](07-storage.md)
- [09 — Implement Secure Login, Logout, and Session Management](09-login-sessions.md)
- [10 — Implement Registration and Email Verification](10-registration.md)
- [11 — Implement Password Change, Recovery, and Administrator Force Reset](11-password-reset.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Define an explicit editable-field allowlist; exclude global authority, project role, account status, and verification state.
2. Implement get-my-profile, update-my-profile, start-email-change, and confirm-email-change for the current user only.
3. Reuse email verification; retain the verified address until the change flow completes.
4. Integrate upload-profile-picture and remove-profile-picture with storage and type/size validation.
5. Create the Account page with identity, email, language, picture, status, and a security/password link.

## Definition of Done — completion checklist

Notion's Acceptance criteria property is empty; the checklist below combines that fact with concrete points grounded in the Users and permissions/API behavior/Data model fields above.

- [ ] `update-my-profile` operates against an explicit allowlist of self-editable fields; Application Administrator authority, project role, account active/archive status, and verification state are rejected even if submitted in the request body (not just hidden in the UI).
- [ ] `get-my-profile`/`update-my-profile` never allow a user to read or modify another user's profile — the target is always the authenticated caller, never an ID taken from client input.
- [ ] Changing email uses `start-email-change` → verification → `confirm-email-change`; the previously verified email stays authoritative and usable for login until the change is confirmed.
- [ ] Profile-picture upload/removal go through the shared storage service ([07](07-storage.md)) with type/size validation; no local filesystem path is ever exposed to the browser.
- [ ] The selected locale/preferred language is persisted and actually changes localized UI on subsequent screens (via the localization resolver mentioned in Behavior).
- [ ] Profile edits never alter project membership, project role, or permissions — verified by checking that a profile update does not touch any `ProjectMembership` record.
- [ ] The Account page surfaces: identity fields, email, language, picture, account verification/status summary (read-only), and a link into the password/security flow ([11](11-password-reset.md)).
- [ ] Automated tests cover: a successful self-update within the allowlist; a rejected attempt to change a privileged field; the full email-change verification flow; and picture upload/removal through the storage abstraction.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

Acceptance criteria are empty in the source; checks are proposed. The explicit cycle with login ([09](09-login-sessions.md) is `Blocked by` this task, while this task is itself blocked by login) is documented in [pending decisions](PENDING-DECISIONS.md) — implement a minimal current-identity/session slice first, then profile, per that guidance.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [12.01 — Define an explicit editable-field allowlist](sub-issues/12-01.md)
- [12.02 — Implement get-my-profile, update-my-profile, start-email-change, and confirm-email-change for the current…](sub-issues/12-02.md)
- [12.03 — Reuse email verification](sub-issues/12-03.md)
- [12.04 — Integrate upload-profile-picture and remove-profile-picture with storage and type/size validation](sub-issues/12-04.md)
- [12.05 — Create the Account page with identity, email, language, picture, status, and a security/password link](sub-issues/12-05.md)
