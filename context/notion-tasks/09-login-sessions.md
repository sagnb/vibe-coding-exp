# 09 — Implement Secure Login, Logout, and Session Management

## Source and usage

- Source: [Implement Secure Login, Logout, and Session Management](https://app.notion.com/p/Implement-Secure-Login-Logout-and-Session-Management-3ab4993602938059950ad70dd8d2e9e9).
- Notion ID: `3ab4993602938059950ad70dd8d2e9e9`.
- Source domain: Authentication & User Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Implement Self-Service Profile and Account Preferences.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Authenticate active users with email and password, establish a server-managed JWT-backed ReLiS session, expose the current user safely, and end sessions on logout or expiry. Project selection is URL-based and never stored as an active-project session value.
- **Functional Domain:** Authentication & User Management
- **Behavior:** Legacy ReLiS offers login/logout and stores the active user and selected project in a CodeIgniter session; passwords are checked through legacy data access. next-relis replaces this with modern password hashing, secure session cookies/tokens, expiration, renewal, and a dedicated current-user API.
- **Users and permissions:** All active users can log in and log out. Archived, disabled, or unverified users receive a localized non-revealing error and cannot receive an authenticated session.
- **UI behavior:**
  - Login form validates credentials, displays localized error states, and redirects authenticated users to their authorized landing page.
  - Logout is available from authenticated navigation and immediately clears client-visible state.
  - Expired sessions redirect to login while preserving a safe return URL.
- **API behavior:**
  - Implement email/password login, logout, session refresh/renewal where applicable, and current user use cases.
  - After successful local authentication, issue a short-lived ReLiS JWT in an HttpOnly, Secure, SameSite=Lax cookie. Validate issuer, audience, signature, expiry, session ID, and revocation/token version on every protected request. Do not store bearer tokens in localStorage or sessionStorage.
  - JWT claims must remain minimal (user subject, session ID, issued/expiry times, issuer, audience, token ID, and token version if needed). They must not contain project roles, permissions, or an active project; the backend resolves authorization from current database state.
  - Protect API routes through authentication middleware; APIs return consistent unauthenticated/expired-session error codes.
  - Rate-limit login attempts, apply CSRF protections to unsafe cookie-authenticated endpoints, and avoid user-enumeration responses.
- **Data model:**
  - Control DB: local user credentials hashed with asynchronous Node.js `crypto.scrypt`, versioned scrypt parameters, a cryptographically random per-credential salt of at least 16 bytes, account status, server session/revocation metadata, and last-login audit data. Compare derived keys in constant time and rehash after successful login when approved parameters change.
  - Control DB / secrets: JWT signing keys must be stored in managed secret/key configuration, not in the database. Include a key ID (`kid`) to support planned key rotation.
  - Project DB: none.
- **Acceptance criteria:**
  - [ ] An active, eligible user can log in, retrieve their current identity, and log out.
  - [ ] Protected API requests reject missing, invalid, revoked, and expired sessions.
  - [ ] Logout invalidates the current session and clears web application state.
  - [ ] No session state stores or selects an active project.
  - [ ] Credentials and account-status errors are localized and do not disclose whether an email exists.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Implement Self-Service Profile and Account Preferences
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Authenticate eligible users and maintain revocable ReLiS sessions, using a short-lived JWT in a secure cookie and URL-based project context.

## Proposed prerequisites

- [03 — Create control Prisma schema](03-control-prisma.md)
- [05 — Implement API app shell](05-api-shell.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Implement Control DB credentials using asynchronous `crypto.scrypt`, versioned parameters, and a random salt of at least 16 bytes; compare derived keys in constant time and rehash after login when needed.
2. Implement login, current user, logout, and renewal where applicable; reject archived, disabled, or unverified accounts.
3. Issue JWTs in an HttpOnly, Secure, SameSite=Lax cookie. Validate signature, issuer, audience, expiry, session ID, and revocation/token version on every protected request.
4. Keep claims minimal: subject, session, issued/expiry times, issuer, audience, token ID, and version if needed; retrieve roles and permissions from the database.
5. Keep signing keys in secret configuration with `kid`; apply rate limiting, CSRF protection, and non-enumerating responses.
6. Create localized screens, safe return paths after expiry, and immediate web-state clearing on logout.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/Data model fields.

- [ ] A user can complete login → retrieve current identity → logout end to end.
- [ ] Passwords are hashed with `crypto.scrypt`, versioned parameters, and a per-credential random salt ≥16 bytes; verification compares in constant time; login triggers a rehash if the configured parameters have since been upgraded.
- [ ] The session JWT is issued in an HttpOnly, Secure, SameSite=Lax cookie only — never returned in a JSON body or storable via `localStorage`/`sessionStorage`.
- [ ] JWT claims are minimal (subject, session ID, issued/expiry, issuer, audience, token ID, token version) and contain **no** project role, permission, or active-project value.
- [ ] Every protected request validates signature, issuer, audience, expiry, session ID, and revocation/token version — a revoked session must be rejected even with an otherwise-valid, non-expired token.
- [ ] Missing, invalid, revoked, and expired sessions all produce a consistent, documented unauthenticated/expired error code (not ad hoc per-route errors).
- [ ] Logout invalidates the server-side session (not just clearing the cookie) and immediately clears client-visible authenticated state.
- [ ] No session state anywhere (cookie, JWT claim, server session record) stores or resolves an "active project" — project context is derived purely from the URL per request.
- [ ] Archived, disabled, or unverified accounts receive a login rejection; the error message is localized and does not reveal whether the submitted email exists in the system.
- [ ] Login attempts are rate-limited, and CSRF protection is applied to unsafe (state-changing) cookie-authenticated endpoints.
- [ ] Signing keys live in managed secret/key configuration (not the database) and carry a `kid` for future rotation.
- [ ] Automated tests cover: successful login/logout; rejected login for archived/disabled/unverified accounts with a non-enumerating message; access with a missing/invalid/expired/revoked token each rejected; and a check that no project data appears in the JWT.
- [ ] Report which commands were run and their actual results.

## Gaps and pending decisions

There is an explicit circular dependency with profile management ([12](12-profile.md)). Proposal: implement current identity and a minimal session first, then profile and integration; retain the cycle as a planning issue.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [09.01 — Implement Control DB credentials using asynchronous crypto](sub-issues/09-01.md)
- [09.02 — Implement login, current user, logout, and renewal where applicable](sub-issues/09-02.md)
- [09.03 — Issue JWTs in an HttpOnly, Secure, SameSite=Lax cookie](sub-issues/09-03.md)
- [09.04 — Keep claims minimal: subject, session, issued/expiry times, issuer, audience, token ID, and version if needed](sub-issues/09-04.md)
- [09.05 — Keep signing keys in secret configuration with kid](sub-issues/09-05.md)
- [09.06 — Create localized screens, safe return paths after expiry, and immediate web-state clearing on logout](sub-issues/09-06.md)
