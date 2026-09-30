# 14 — Implement Google OpenID Connect Sign-In

## Source and usage

- Source: [Implement Google OpenID Connect Sign-In](https://app.notion.com/p/Implement-Google-OpenID-Connect-Sign-In-3ad49936029380e283afd44fe89368d3).
- Notion ID: `3ad49936029380e283afd44fe89368d3`.
- Source domain: Authentication & User Management.
- Source status on 2026-09-20: Not Started; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Not Started
- **Description:** Add optional "Continue with Google" authentication through Google OpenID Connect (OIDC). This ticket is separate from local email/password login: it delegates ReLiS-session creation, logout, JWT validation, and session revocation to `RELIS-T-USER-002` (functionally, the login task, [09](09-login-sessions.md)). Google proves identity; it never grants an application role, a project membership, or an active-project selection.
- **Functional Domain:** Authentication & User Management
- **Behavior:** Legacy ReLiS has no Google sign-in. ReLiS Next currently has the Hono API shell only and contains no authentication, OIDC, OAuth callback, JWT, or external-identity implementation.
- **Users and permissions:**
  - An active ReLiS user with an explicitly linked Google identity can use either local email/password login or Google sign-in.
  - Google authentication identifies a person only. Application Administrator status, Project Admin status, memberships, and project roles continue to be resolved from the control database for every protected request.
  - Never automatically link an existing ReLiS account merely because its email matches a Google email. Linking must be explicit and confirmed by an authenticated ReLiS account holder or performed by an Application Administrator.
  - Disabled, archived, or otherwise ineligible accounts cannot receive a ReLiS session after Google authentication.
- **UI behavior:**
  - Show a localized "Continue with Google" button on the login page only when Google OIDC is enabled in application configuration.
  - Opening the Google flow preserves only a validated same-origin return path. After the callback, return the user to that path or to their authorized landing page.
  - Present localized, non-revealing error states for cancelled, expired, invalid, or not-authorized Google sign-in attempts. Do not expose provider token or claim details.
  - Provide an explicit account-link/unlink interaction in authenticated account settings. Linking requires recent ReLiS authentication and confirmation; unlinking is blocked when it would leave the user with no approved login method.
  - Local email/password login remains visible and fully supported; Google sign-in is an additional method, not a replacement for Project Admin-provisioned local accounts.
- **API behavior:**
  - Provide a Google authorization-start use case and an approved redirect callback using the OIDC authorization-code flow with PKCE. Register only approved production and development redirect URIs with Google.
  - Create a short-lived, single-use login transaction with a cryptographically random state, nonce, PKCE verifier, validated return path, creation time, expiry, and consumed status.
  - At callback, validate the transaction before exchanging the code. Validate the ID token signature through Google's keys plus issuer, audience, authorized party when applicable, expiry, nonce, and subject (`sub`). Use only `openid email profile` scopes for this feature.
  - Identify a Google identity from its immutable `sub`, not its email address. Require the email to be provider-verified before it can support an allowed self-registration path.
  - Do not reuse a Google ID token or access token as a ReLiS session. Do not request, return, persist, or log Google access/refresh tokens for this feature.
  - On success, call the shared session issuer from `RELIS-T-USER-002` to issue the normal short-lived ReLiS JWT in its secure HttpOnly cookie. The JWT must contain no project role, permission snapshot, or active project.
  - Reject replayed or expired callback transactions and invalid tokens. Rate-limit authorization start and callback failures, and apply the established CSRF/CORS policy from `RELIS-T-USER-002`.
- **Data model:**
  - Control DB: add an external-identity record with a unique `(provider, provider_subject)` pair. For Google, `provider = google` and `provider_subject` = the validated OIDC `sub`. Store only the minimum safe metadata required for account linking and audit, such as current verified email and link timestamps.
  - Control DB: persist a short-lived, single-use OIDC login transaction. Store `state` as a digest; store the PKCE verifier only in encrypted/protected server-side state; retain nonce, return path, expiry, and consumption time. Remove or expire transactions promptly.
  - Control DB: use the shared user, account-status, session, token-version, and audit records defined by `RELIS-T-USER-002`; do not duplicate session/JWT tables for Google.
  - Secret configuration: Google client ID, client secret when applicable, redirect URIs, and permitted configuration belong in managed environment/secret configuration, never in source control or the Project DB.
  - Project DB: none.
- **Acceptance criteria:**
  - [ ] Google OIDC can be enabled or disabled globally; the login UI shows the Google action only when enabled.
  - [ ] A valid Google authorization-code flow with PKCE produces the same ReLiS JWT session as `RELIS-T-USER-002`, and local email/password authentication continues to work independently.
  - [ ] Invalid, expired, replayed, or mismatched `state`, nonce, PKCE, signature, issuer, audience, expiry, or subject claims are rejected without a session.
  - [ ] The external identity is keyed by Google's `sub`; email matching alone never silently links accounts or grants access.
  - [ ] Google sign-in never creates an Application Administrator, Project Admin, project membership, or project role. Any permitted new account follows the global registration mode and is initially unprivileged.
  - [ ] Provider tokens and OIDC secrets are neither sent to the browser nor stored or written to logs by this feature.
  - [ ] An account can unlink Google only when it retains another approved sign-in method.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Add optional Google authentication using the existing ReLiS session issuer.

## Proposed prerequisites

- [09 — Implement Secure Login, Logout, and Session Management](09-login-sessions.md)
- [10 — Implement Registration and Email Verification](10-registration.md)
- [12 — Implement Self-Service Profile and Account Preferences](12-profile.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Configure enablement and approved redirects; keep local login available.
2. Implement authorization code + PKCE with a short-lived, single-use transaction: state digest, nonce, protected verifier, same-origin return path, expiry, and consumption.
3. Validate the transaction before code exchange; validate signature using Google's keys, issuer, audience, `azp` when applicable, expiry, nonce, and `sub`; use only `openid`/`email`/`profile`.
4. Identify links by `(provider, sub)`, never automatically link by email; require confirmation with recent authentication or an administrator action.
5. Issue the shared ReLiS session without project roles/context in the JWT; do not persist, log, or return Google tokens or reuse them as a session.
6. Implement link/unlink and block removal of the last approved sign-in method; apply global policy to any verified, unprivileged self-registration.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/Data model fields.

- [ ] Google OIDC has a single global enable/disable switch; the "Continue with Google" button only renders when enabled, and the callback route itself rejects requests when disabled.
- [ ] The full authorization-code + PKCE flow is implemented: `state` (stored as a digest), `nonce`, PKCE verifier (protected server-side state only), same-origin return path, creation time, expiry, and single-use consumption of the login transaction.
- [ ] The callback validates, in order: the transaction (not expired/not already consumed/matching state), then the ID token — signature via Google's JWKS, issuer, audience, `azp` where applicable, expiry, nonce, and `sub` — using only `openid email profile` scopes.
- [ ] The external identity is looked up/created by `(provider, sub)` — never by matching on email alone; an email match with an existing local account requires an explicit, confirmed linking step (by the authenticated account holder with recent auth, or by an administrator).
- [ ] A successful Google sign-in issues exactly the same kind of ReLiS session/JWT as local login ([09](09-login-sessions.md)) — same cookie flags, same minimal-claims shape, no project role/permission/active-project embedded.
- [ ] Google ID/access/refresh tokens are never persisted, logged, or returned to the browser; only the minimal external-identity metadata (provider, subject, verified email, link timestamps) is stored.
- [ ] A brand-new account created via Google sign-in (when self-registration is permitted) starts unprivileged and follows the same global registration-mode policy as [10](10-registration.md) — it is never auto-granted Application Administrator, Project Admin, or any project membership.
- [ ] Unlink is blocked when it would leave the account with zero approved sign-in methods (e.g., no local password set and Google is the only linked method).
- [ ] Automated tests cover: a valid flow producing a session; rejection of tampered/replayed `state`/`nonce`; rejection of an ID token with wrong issuer/audience/expired signature; and the unlink-guard behavior.
- [ ] Report which commands were run and their actual results, and note that the source's `RELIS-T-USER-002` identifier refers functionally to [09 — Secure Login, Logout, and Session Management](09-login-sessions.md) (see Gaps below).

## Gaps and pending decisions

`RELIS-T-USER-002` functionally refers to the login task ([09](09-login-sessions.md)); this identifier was not exposed as a property on the consulted pages.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [14.01 — Configure enablement and approved redirects](sub-issues/14-01.md)
- [14.02 — Implement authorization code + PKCE with a short-lived, single-use transaction: state digest, nonce,…](sub-issues/14-02.md)
- [14.03 — Validate the transaction before code exchange](sub-issues/14-03.md)
- [14.04 — Identify links by (provider, sub), never automatically link by email](sub-issues/14-04.md)
- [14.05 — Issue the shared ReLiS session without project roles/context in the JWT](sub-issues/14-05.md)
- [14.06 — Implement link/unlink and block removal of the last approved sign-in method](sub-issues/14-06.md)
