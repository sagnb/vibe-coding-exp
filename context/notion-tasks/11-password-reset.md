# 11 — Implement Password Change, Recovery, and Administrator Force Reset

## Source and usage

- Source: [Implement Password Change, Recovery, and Administrator Force Reset](https://app.notion.com/p/Implement-Password-Change-Recovery-and-Administrator-Force-Reset-3ac499360293806fad8dec50174659d7).
- Notion ID: `3ac499360293806fad8dec50174659d7`.
- Source domain: Authentication & User Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Enable users to change their password and recover a forgotten password securely. Allow Application Administrators to force a reset without seeing, setting, or receiving the user's password.
- **Functional Domain:** Authentication & User Management
- **Behavior:** Legacy ReLiS provides password-management functions. The new relis uses time-limited, single-use reset tokens and delegates all mail delivery to the localized account-email adapter.
- **Users and permissions:**
  - Authenticated users change their own password after confirming their current password.
  - Visitors request password recovery using a non-enumerating email form.
  - Administrators force a password-reset flow for a user; they never set a user's password directly.
- **UI behavior:**
  - Provide change-password, forgot-password, reset-password, and forced-reset confirmation pages.
  - A forced reset informs the administrator of the result without disclosing token or password data.
  - A successful reset revokes existing active sessions for that account. It does not set a future password-expiration date or require periodic rotation.
- **API behavior:**
  - Provide change-password, request-password-reset, validate-reset-token, complete-reset, and administrator-force-reset use cases.
  - Generate, hash, expire, and consume reset tokens securely.
  - Revoke sessions after any successful password change/reset.
- **Data model:** Control DB: password hash, password-reset token digest/expiry/consumption metadata, force-reset flag where needed, and session revocation metadata. Project DB: none.
- **Acceptance criteria:**
  - [ ] A user can securely change their password after current-password confirmation.
  - [ ] A password-reset request returns the same response for existing and unknown email addresses.
  - [ ] Reset tokens expire, are single-use, and are stored only as a digest.
  - [ ] A successful reset revokes all prior sessions for that account.
  - [ ] Administrators can force a reset but cannot read or set another user's password directly.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Implement password change, recovery, and forced reset without allowing administrators to view or set other users' passwords.

## Proposed prerequisites

- [09 — Implement Secure Login, Logout, and Session Management](09-login-sessions.md)
- [10 — Implement Registration and Email Verification](10-registration.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Implement change-password with current-password confirmation and the same hashing parameters as login.
2. Implement request-password-reset, validate-reset-token, and complete-reset with digest storage, expiry, and single-use consumption.
3. Offer administrator-force-reset only to authorized administrators; never return a password or token to the administrator.
4. Revoke previous sessions after every successful password change/reset.
5. Create localized change, forgotten-password, reset, and administrator-confirmation screens; do not add periodic password expiry.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the UI behavior/Data model fields.

- [ ] `change-password` requires and verifies the user's current password before accepting a new one, using the same hashing parameters/pipeline as login ([09](09-login-sessions.md)).
- [ ] `request-password-reset` returns an identical response regardless of whether the submitted email matches an existing account (no enumeration).
- [ ] Reset tokens are random, single-use, time-limited, and stored only as a digest — never plain text — consistent with the registration-token pattern in [10](10-registration.md).
- [ ] `administrator-force-reset` triggers the normal reset flow for the target user; the administrator's response never contains the reset token, a link with the token embedded and readable in a UI they see verbatim, or the resulting password.
- [ ] A successful password change or reset revokes **all** existing active sessions for that account (not just the one used to perform the change).
- [ ] No periodic password-expiration or forced-rotation mechanism is introduced — the source explicitly excludes it.
- [ ] Localized UI exists for: change-password, forgot-password, reset-password, and a forced-reset confirmation for the administrator.
- [ ] Automated tests cover: successful change with correct current password; rejected change with wrong current password; full reset flow (request → token → complete) with session revocation verified; and identical request-reset responses for existing vs. unknown emails.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

No specific contradiction is highlighted in this summary. Cross-cutting decisions in the pending-decisions register still apply. Empty source fields do not authorize expanding scope.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [11.01 — Implement change-password with current-password confirmation and the same hashing parameters as login](sub-issues/11-01.md)
- [11.02 — Implement request-password-reset, validate-reset-token, and complete-reset with digest storage, expiry,…](sub-issues/11-02.md)
- [11.03 — Offer administrator-force-reset only to authorized administrators](sub-issues/11-03.md)
- [11.04 — Revoke previous sessions after every successful password change/reset](sub-issues/11-04.md)
- [11.05 — Create localized change, forgotten-password, reset, and administrator-confirmation screens](sub-issues/11-05.md)
