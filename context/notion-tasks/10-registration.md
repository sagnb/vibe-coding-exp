# 10 — Implement Registration and Email Verification

## Source and usage

- Source: [Implement Registration and Email Verification](https://app.notion.com/p/Implement-Registration-and-Email-Verification-48f4993602938265b06681ed2318e707).
- Notion ID: `48f4993602938265b06681ed2318e707`.
- Source domain: Authentication & User Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Allow public, disabled registration according to global settings, and require email verification before access to any functionality. Include the anti-abuse challenge.
- **Functional Domain:** Authentication & User Management
- **Behavior:** Legacy ReLiS supports user registration, captcha validation, confirmation codes, and cleanup of unvalidated accounts (after one hour?). NeoReLiS keeps the account-lifecycle intent with expiring, single-use verification tokens and localized account emails.
- **Users and permissions:**
  - Visitors can register.
  - Administrators can create users through the separate admin ticket.
  - Unverified accounts cannot access protected project features.
- **UI behavior:**
  - Registration page adapts to registration mode: available, disabled, or admin-managed.
  - Display a verification-pending screen with resend-verification action that does not reveal account existence.
  - Display the anti-abuse challenge.
- **API behavior:**
  - Provide register, verify-email, and resend-verification use cases.
  - Generate secure, single-use, expiring verification tokens; invalidate prior tokens when a new one is issued.
  - Send localized account-verification email through a shared mail adapter.
  - Apply abuse protection to registration, resend, and verification attempts.
- **Data model:** Control DB: user account status, verification token digest, expiry, consumed timestamp, retry metadata, and registration audit data. File storage / Project DB: none.
- **Acceptance criteria:**
  - [ ] Registration can be configured as public, disabled, or administrator-only.
  - [ ] A new public-registration account remains unable to access protected features until verified.
  - [ ] Verification tokens are expiring, single-use, and never stored in plain text.
  - [ ] Resend and registration endpoints are abuse-protected and do not enable account enumeration.
  - [ ] The optional anti-abuse challenge appears only when public registration policy requires it.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Provide configurable registration and email verification before protected access.

## Proposed prerequisites

- [03 — Create control Prisma schema](03-control-prisma.md)
- [05 — Implement API app shell](05-api-shell.md)
- [06 — Complete job queue and progress platform](06-job-platform.md)
- [09 — Implement Secure Login, Logout, and Session Management](09-login-sessions.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Define public, disabled, and administrator-managed modes according to global configuration.
2. Implement register, verify-email, and resend-verification with abuse controls.
3. Generate random, single-use tokens with digest, expiry, and consumption metadata in the Control DB; invalidate previous tokens on resend.
4. Integrate the shared localized email adapter; unverified accounts remain ineligible for protected features.
5. Create registration and verification-pending screens, non-enumerating resend, and an anti-abuse challenge when policy requires it.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/Data model fields.

- [ ] Registration mode (public / disabled / administrator-only) is enforced both in the UI (which form/state is shown) and in the API (a direct call cannot bypass a "disabled" or "administrator-only" configuration).
- [ ] A newly registered public account cannot access any protected project feature until its email is verified.
- [ ] Verification tokens are: random and single-use; stored only as a digest (never plain text); time-limited; and issuing a new token invalidates the previous one.
- [ ] `resend-verification` and `register` never reveal whether a given email already has an account — identical responses for existing vs. unknown emails.
- [ ] Registration, resend, and verification endpoints are rate-limited/abuse-protected (e.g., per-IP and per-email throttling).
- [ ] The anti-abuse challenge (e.g. CAPTCHA) is shown only when the public-registration policy requires it — it is not unconditionally mandatory (resolve, don't silently pick one interpretation — see Gaps below).
- [ ] Verification emails are sent through the shared localized mail adapter (not a bespoke sender), and are localized to the user's chosen/detected language.
- [ ] Automated tests cover: successful registration + verification; access-denial for an unverified account attempting a protected action; expired/consumed token rejection; and identical resend responses for existing vs. unknown email.
- [ ] Report which commands were run and their actual results, and record which anti-abuse provider/approach was chosen (or that none was integrated yet) as an explicit decision.

## Gaps and pending decisions

The description requires an anti-abuse challenge, while acceptance criteria make it optional depending on policy. Do not select a provider or make it globally mandatory without defining that policy.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [10.01 — Define public, disabled, and administrator-managed modes according to global configuration](sub-issues/10-01.md)
- [10.02 — Implement register, verify-email, and resend-verification with abuse controls](sub-issues/10-02.md)
- [10.03 — Generate random, single-use tokens with digest, expiry, and consumption metadata in the Control DB](sub-issues/10-03.md)
- [10.04 — Integrate the shared localized email adapter](sub-issues/10-04.md)
- [10.05 — Create registration and verification-pending screens, non-enumerating resend, and an anti-abuse challenge…](sub-issues/10-05.md)
