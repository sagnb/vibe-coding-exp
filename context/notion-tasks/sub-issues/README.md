# Local sub-issues — Notion task decomposition

## Status and provenance

264 local draft sub-issues expand all 264 numbered steps from the 42 parent guides. These are implementation proposals derived from the English Notion adaptations, **not newly created Notion cards**, verbatim exports, approved product decisions, or completed work. Original Notion links and parent step numbers remain attached to every sub-issue.

The parent source snapshot is dated 2026-09-20. On 2026-09-27 the board and runtime-config/QA-results requirements were consulted again; the runtime-config empty acceptance field and QA exclusion/restoration contradiction were confirmed. This is not a fresh full-property export of all 42 pages. Consult the live source before implementing a selected issue.

## How to execute one issue

Read the [parent index](../README.md), [pending decisions](../PENDING-DECISIONS.md), [required structure](../../project-structure.md), [stack](../../stack.yml), and [testing policy](../../testing-policy.md). The parent describes the whole feature; each child limits the selected delivery. Numbers preserve the existing proposed sequence, not Notion priority or mandatory dependency edges.

Select one issue explicitly in the user prompt. Inspect prerequisite outputs, implement only the authorized slice, test every new behavior, diagnose/fix/rerun failures, and report evidence. Do not implement all prerequisites or siblings automatically. Source contradictions block only the affected behavior; ask for a decision and continue independent authorized work. Missing source acceptance criteria remain proposed engineering checks, not attributed to Notion.

The scenario checks in each issue supplement (not replace) the full source criteria. Convert them into concrete automated assertions for code changes. For inventory/decision/documentation-only work, produce verifiable evidence without introducing unrelated features just to create tests. A child can be completed without declaring its parent complete; all parent criteria and real integrations still need verification.

## Inventory

### 01 — Implement runtime config validation

Parent: [01-runtime-config.md](../01-runtime-config.md).

| Local issue | Scope (original step) |
| --- | --- |
| [01.01](01-01.md) | Inventory the variables used by each process and separate public values from secrets |
| [01.02](01-02.md) | Define the shared configuration contract and required validation |
| [01.03](01-03.md) | Integrate validated configuration loading into each process startup, before connections and writes |
| [01.04](01-04.md) | Expose health/readiness diagnostics by configuration category without returning values |
| [01.05](01-05.md) | Document variables and examples without real credentials |

### 02 — Add Docker Compose local stack

Parent: [02-docker-compose.md](../02-docker-compose.md).

| Local issue | Scope (original step) |
| --- | --- |
| [02.01](02-01.md) | Check which applications exist and their actual commands |
| [02.02](02-02.md) | Define services and shared configuration, including the Control DB and per-project test databases |
| [02.03](02-03.md) | Configure local persistent volumes and exclude them from version control |
| [02.04](02-04.md) | Expose health/readiness through the local proxy and restrict database and storage port exposure |
| [02.05](02-05.md) | Document startup, diagnostics, and local access to MailHog/Mailpit and job observability |

### 03 — Create control Prisma schema

Parent: [03-control-prisma.md](../03-control-prisma.md).

| Local issue | Scope (original step) |
| --- | --- |
| [03.01](03-01.md) | Locate @relis/database in the target repository |
| [03.02](03-02.md) | Consolidate identity, session, project, membership, invitation, draft, job, and audit contracts from the… |
| [03.03](03-03.md) | Resolve the PROJECT_OWNER/PROJECT_ADMIN discrepancy before finalizing identity and membership enums and… |
| [03.04](03-04.md) | Create the PostgreSQL schema, static migrations, and Control DB Prisma client |
| [03.05](03-05.md) | Expose typed repositories |

### 04 — Implement reusable project Prisma schema

Parent: [04-project-prisma.md](../04-project-prisma.md).

| Local issue | Scope (original step) |
| --- | --- |
| [04.01](04-01.md) | Inventory the Paper, protocol, assignment, screening, QA, classification, reporting, and audit contracts… |
| [04.02](04-02.md) | Extend the existing database package with a schema and static migrations separate from the Control DB |
| [04.03](04-03.md) | Represent categories, dependencies, and subcategories as records in fixed models, without generating… |
| [04.04](04-04.md) | Create typed repositories and schema version/compatibility checks |
| [04.05](04-05.md) | Validate the same schema in two independent databases and prepare it for the provisioning template |

### 05 — Implement API app shell

Parent: [05-api-shell.md](../05-api-shell.md).

| Local issue | Scope (original step) |
| --- | --- |
| [05.01](05-01.md) | Verify the stated baseline: Hono, CORS, GET /, and GET /health |
| [05.02](05-02.md) | Organize route registration/versioning, contracts, and input validation |
| [05.03](05-03.md) | Add request IDs, structured logs, and error mapping for validation, authentication, authorization, not… |
| [05.04](05-04.md) | Define authentication, policy, and project-context integration points |
| [05.05](05-05.md) | Separate health from readiness |

### 06 — Complete job queue and progress platform

Parent: [06-job-platform.md](../06-job-platform.md).

| Local issue | Scope (original step) |
| --- | --- |
| [06.01](06-01.md) | Verify apps/worker and the pg-boss dependency mentioned in Notion |
| [06.02](06-02.md) | Implement enqueueJob, getJobStatus, listJobs, retryJob, cancelJob, and handler registration |
| [06.03](06-03.md) | Keep pg-boss tables in the Control DB and metadata for actor, project, progress, safe error, correlation… |
| [06.04](06-04.md) | Carry opaque IDs and references in payloads |
| [06.05](06-05.md) | Create the Jobs page and queued/running/completed/failed/cancelled status component |
| [06.06](06-06.md) | Validate retry/cancel and worker failures without duplicating effects in consuming tasks |

### 07 — Implement upload/storage service

Parent: [07-storage.md](../07-storage.md).

| Local issue | Scope (original step) |
| --- | --- |
| [07.01](07-01.md) | Define ownership by user/project and policies supplied by each feature |
| [07.02](07-02.md) | Implement createUploadIntent, completeUpload, scanAndValidateUpload, getArtifact, downloadArtifact,… |
| [07.03](07-03.md) | Validate type, size, checksum, and malware policy before making an object available |
| [07.04](07-04.md) | Persist lifecycle references in the Control DB and feature ownership in the Project DB as appropriate |
| [07.05](07-05.md) | Create reusable upload states and artifact cards with expiry and authorized downloads |
| [07.06](07-06.md) | Integrate isolation, revocation, and safe failure handling |

### 08 — Seed installation and demo data

Parent: [08-installation-demo.md](../08-installation-demo.md).

| Local issue | Scope (original step) |
| --- | --- |
| [08.01](08-01.md) | Implement getInstallationStatus, validateInstallationEnvironment, and initializeControlPlane with guards… |
| [08.02](08-02.md) | Apply the Control DB schema and create the first Application Administrator, required settings, and… |
| [08.03](08-03.md) | Create a protected first-run screen with validation, administrator creation, progress, and one-time completion |
| [08.04](08-04.md) | Integrate seedOptionalDemoData only after the normal creation/provisioning flow in guides 16 and 18 is ready |
| [08.05](08-05.md) | Create demo data in a separate Project DB, clearly marked as non-production, and audit setup |

### 09 — Implement Secure Login, Logout, and Session Management

Parent: [09-login-sessions.md](../09-login-sessions.md).

| Local issue | Scope (original step) |
| --- | --- |
| [09.01](09-01.md) | Implement Control DB credentials using asynchronous crypto |
| [09.02](09-02.md) | Implement login, current user, logout, and renewal where applicable |
| [09.03](09-03.md) | Issue JWTs in an HttpOnly, Secure, SameSite=Lax cookie |
| [09.04](09-04.md) | Keep claims minimal: subject, session, issued/expiry times, issuer, audience, token ID, and version if needed |
| [09.05](09-05.md) | Keep signing keys in secret configuration with kid |
| [09.06](09-06.md) | Create localized screens, safe return paths after expiry, and immediate web-state clearing on logout |

### 10 — Implement Registration and Email Verification

Parent: [10-registration.md](../10-registration.md).

| Local issue | Scope (original step) |
| --- | --- |
| [10.01](10-01.md) | Define public, disabled, and administrator-managed modes according to global configuration |
| [10.02](10-02.md) | Implement register, verify-email, and resend-verification with abuse controls |
| [10.03](10-03.md) | Generate random, single-use tokens with digest, expiry, and consumption metadata in the Control DB |
| [10.04](10-04.md) | Integrate the shared localized email adapter |
| [10.05](10-05.md) | Create registration and verification-pending screens, non-enumerating resend, and an anti-abuse challenge… |

### 11 — Implement Password Change, Recovery, and Administrator Force Reset

Parent: [11-password-reset.md](../11-password-reset.md).

| Local issue | Scope (original step) |
| --- | --- |
| [11.01](11-01.md) | Implement change-password with current-password confirmation and the same hashing parameters as login |
| [11.02](11-02.md) | Implement request-password-reset, validate-reset-token, and complete-reset with digest storage, expiry,… |
| [11.03](11-03.md) | Offer administrator-force-reset only to authorized administrators |
| [11.04](11-04.md) | Revoke previous sessions after every successful password change/reset |
| [11.05](11-05.md) | Create localized change, forgotten-password, reset, and administrator-confirmation screens |

### 12 — Implement Self-Service Profile and Account Preferences

Parent: [12-profile.md](../12-profile.md).

| Local issue | Scope (original step) |
| --- | --- |
| [12.01](12-01.md) | Define an explicit editable-field allowlist |
| [12.02](12-02.md) | Implement get-my-profile, update-my-profile, start-email-change, and confirm-email-change for the current… |
| [12.03](12-03.md) | Reuse email verification |
| [12.04](12-04.md) | Integrate upload-profile-picture and remove-profile-picture with storage and type/size validation |
| [12.05](12-05.md) | Create the Account page with identity, email, language, picture, status, and a security/password link |

### 13 — Implement Administrator User Directory and Account Lifecycle Management

Parent: [13-admin-users.md](../13-admin-users.md).

| Local issue | Scope (original step) |
| --- | --- |
| [13.01](13-01.md) | Implement list-users, get-user, create-user, update-user-admin, and global-authority assignment for… |
| [13.02](13-02.md) | Protect the last active administrator from archival or authority removal |
| [13.03](13-03.md) | Implement archive/restore without permanent deletion |
| [13.04](13-04.md) | Integrate force-password-reset with the existing secure flow |
| [13.05](13-05.md) | Create a searchable, paginated directory, details, memberships, status, verification, history, and… |
| [13.06](13-06.md) | Audit actions and preserve historical IDs in project records |

### 14 — Implement Google OpenID Connect Sign-In

Parent: [14-google-oidc.md](../14-google-oidc.md).

| Local issue | Scope (original step) |
| --- | --- |
| [14.01](14-01.md) | Configure enablement and approved redirects |
| [14.02](14-02.md) | Implement authorization code + PKCE with a short-lived, single-use transaction: state digest, nonce,… |
| [14.03](14-03.md) | Validate the transaction before code exchange |
| [14.04](14-04.md) | Identify links by (provider, sub), never automatically link by email |
| [14.05](14-05.md) | Issue the shared ReLiS session without project roles/context in the JWT |
| [14.06](14-06.md) | Implement link/unlink and block removal of the last approved sign-in method |

### 15 — Implement Project Context, Roles, and Membership

Parent: [15-membership.md](../15-membership.md).

| Local issue | Scope (original step) |
| --- | --- |
| [15.01](15-01.md) | Define middleware that verifies the URL projectId, state, membership, and role |
| [15.02](15-02.md) | Implement Team, exact normalized username/email lookup, and adding existing accounts |
| [15.03](15-03.md) | Implement invitations with a fixed role, random token stored as a hash, expiry, and asynchronous email |
| [15.04](15-04.md) | Accept an invitation only with verified ownership of its email |
| [15.05](15-05.md) | Prevent duplicate active memberships and invitations |
| [15.06](15-06.md) | Implement role changes, removal, and restoration while preserving historical work |
| [15.07](15-07.md) | Keep identities, credentials, invitations, memberships, and roles only in the Control DB |

### 16 — Prepare and Provision an Isolated Project Database Template

Parent: [16-project-template.md](../16-project-template.md).

| Local issue | Scope (original step) |
| --- | --- |
| [16.01](16-01.md) | Prepare relis_project_template outside the creation flow, with a complete schema and no project data,… |
| [16.02](16-02.md) | Implement idempotent requests and status in the Control DB |
| [16.03](16-03.md) | In the worker, clone or reuse the database already created for the same project and verify… |
| [16.04](16-04.md) | Do not run migrate deploy, db push, or schema changes during creation |
| [16.05](16-05.md) | Record template release, job, safe failures, timestamps, and audit events |
| [16.06](16-06.md) | Keep the project in provisioning until the wizard persists v1 and activates the membership/project |

### 17 — Implement project database resolver

Parent: [17-project-resolver.md](../17-project-resolver.md).

| Local issue | Scope (original step) |
| --- | --- |
| [17.01](17-01.md) | Locate the existing database/platform package and the Control DB project registry |
| [17.02](17-02.md) | Implement resolveProjectDatabaseClient using only an internal, previously authorized projectId |
| [17.03](17-03.md) | Verify readiness and compatibility before accessing the Project DB |
| [17.04](17-04.md) | Implement getProjectWorkspaceStatus and invalidateProjectClientCache |
| [17.05](17-05.md) | Define client lifecycle/cache behavior and return safe unavailability when the workspace is not ready |

### 18 — Build the Native Project Protocol Wizard

Parent: [18-protocol-wizard.md](../18-protocol-wizard.md).

| Local issue | Scope (original step) |
| --- | --- |
| [18.01](18-01.md) | Define a versioned Zod contract shared by Control DB JSONB drafts, JSON import, step validation, and final… |
| [18.02](18-02.md) | Implement an empty start or validated JSON that only prefills the wizard |
| [18.03](18-03.md) | Build identity fields: title, unique short name, description, and ordered questions |
| [18.04](18-04.md) | Build Screening: reviewer count, decision/criteria conflicts, unanimity/majority, criteria,… |
| [18.05](18-05.md) | Build QA with ordered questions, shared response scale, scores, and threshold |
| [18.06](18-06.md) | Build Simple (boolean/integer/real/date/string/text), List (at least two options), DynamicList, and… |
| [18.07](18-07.md) | Build Simple/Compare reports referencing valid categories and chart types |
| [18.08](18-08.md) | Implement autosave/resume/discard, final review, field errors, and confirmation |
| [18.09](18-09.md) | In a Control DB transaction, reserve a PROVISIONING project, createdByUserId, a protected pending… |
| [18.10](18-10.md) | Orchestrate template cloning, normalized v1 protocol CRUD, and activation only after success |
| [18.11](18-11.md) | Export portable configuration without internal IDs/runtime data/secrets |

### 19 — Implement Project Lifecycle, Project Cards, and Metadata Overview

Parent: [19-project-lifecycle.md](../19-project-lifecycle.md).

| Local issue | Scope (original step) |
| --- | --- |
| [19.01](19-01.md) | Create /projects with My projects (including drafts), My participating projects, and Public projects… |
| [19.02](19-02.md) | Implement title/owner search, Draft/Provisioning/Active/Published/Archived filters, and… |
| [19.03](19-03.md) | Return title, short name, owner, user profile, status, dates, and allowed actions |
| [19.04](19-04.md) | Implement details and editing for title, description, and research questions |
| [19.05](19-05.md) | Implement publish/unpublish and archive/restore with valid transitions, confirmation, and audit |
| [19.06](19-06.md) | Return safe provisioning failures and public access only according to publication policy |

### 20 — Implement Project Workspace Navigation and Home Dashboard

Parent: [20-project-workspace.md](../20-project-workspace.md).

| Local issue | Scope (original step) |
| --- | --- |
| [20.01](20-01.md) | Integrate the authenticated global/project shell with branding, collapsible navigation, URL-based… |
| [20.02](20-02.md) | Provide a shell endpoint with identity, lifecycle, membership, isProjectOwner, global authority, modules,… |
| [20.03](20-03.md) | Create global Dashboard/Projects navigation and project Home, Papers/Imports/Authors/Venues, Screening,… |
| [20.04](20-04.md) | Create Home with breadcrumb, title/description/state, owner card with avatar/name/email, and participants… |
| [20.05](20-05.md) | Display the configured Review workflow phases, QA, and Classification |
| [20.06](20-06.md) | Switching projects invalidates visual/data context and navigates by URL |

### 21 — Implement Direct Project Settings Management

Parent: [21-project-settings.md](../21-project-settings.md).

| Local issue | Scope (original step) |
| --- | --- |
| [21.01](21-01.md) | Reuse wizard contracts/controls and organize General, Screening, QA, Data extraction, and Reporting |
| [21.02](21-02.md) | Apply the detailed UI/API matrix: General for Owner/Admin |
| [21.03](21-03.md) | Implement inline Add/Edit/Save/Cancel/confirmed Delete |
| [21.04](21-04.md) | Manage questions, sources/strategies, and JSON export |
| [21.05](21-05.md) | Manage screening defaults, criteria, and phases |
| [21.06](21-06.md) | Allow rule changes after work starts with an impact warning and confirmation, preserving… |
| [21.07](21-07.md) | Manage QA, categories/subcategories/lists/dependencies, and charts through fixed-schema CRUD |
| [21.08](21-08.md) | Keep createdBy/createdAt/modifiedBy backend-managed |
| [21.09](21-09.md) | Audit mutations and return field errors |

### 22 — Implement audit and undo UI

Parent: [22-audit-undo.md](../22-audit-undo.md).

| Local issue | Scope (original step) |
| --- | --- |
| [22.01](22-01.md) | Define a shared envelope: actor, scope, project, action, target, outcome, correlation ID, timestamp,… |
| [22.02](22-02.md) | Integrate recordAuditEvent into sensitive Control/Project DB mutations |
| [22.03](22-03.md) | Implement searchAuditEvents, getAuditEventDetail, and exportAuditEvents with filters and audited global or… |
| [22.04](22-04.md) | Create OperationBatch/CompensatingAction and explicitly registered handlers |
| [22.05](22-05.md) | Implement previewCompensatingAction/executeCompensatingAction with a reason, revalidation, and a… |
| [22.06](22-06.md) | Create Audit log and Undo centre |

### 23 — Implement system settings and custom labels

Parent: [23-system-settings.md](../23-system-settings.md).

| Local issue | Scope (original step) |
| --- | --- |
| [23.01](23-01.md) | Define a settings registry with type, scope, default, validation, sensitivity, restart requirement, and… |
| [23.02](23-02.md) | Implement getSystemSettings, validateSystemSettingsDraft, updateSystemSettings, and… |
| [23.03](23-03.md) | Separate global settings/revisions in the Control DB from protocol settings/labels in the Project DB |
| [23.04](23-04.md) | Implement listCustomLabels, updateCustomLabelDraft, and publishCustomLabelRevision using stable IDs and… |
| [23.05](23-05.md) | Implement getHelpContent, saveHelpContentDraft, publishHelpContent, and restoreSettingsOrContentRevision… |
| [23.06](23-06.md) | Create settings, labels, and help screens by role |

### 24 — Implement Project Paper List

Parent: [24-paper-list.md](../24-paper-list.md).

| Local issue | Scope (original step) |
| --- | --- |
| [24.01](24-01.md) | Implement queries against the authorized Project DB, with pagination and search by key, title, author,… |
| [24.02](24-02.md) | Filter by Pending (default), Included, or Excluded decisions and return pagination metadata |
| [24.03](24-03.md) | Expose only id/key/title/URL/Source/decision and allowed actions in the list |
| [24.04](24-04.md) | Create the Paper key \| Title \| URL \| Source \| Decision \| Actions table |
| [24.05](24-05.md) | Show ordered authors, DOI, URL, Source, Strategy, Venue, year, and abstract in details |
| [24.06](24-06.md) | Expose Import/Edit/Delete only to Owner/Manager when their respective flows exist |

### 25 — Implement Authors and Affiliations Directory

Parent: [25-authors-affiliations.md](../25-authors-affiliations.md).

| Local issue | Scope (original step) |
| --- | --- |
| [25.01](25-01.md) | Model/reuse Author, Affiliation, and ordered PaperAuthor records in the Project DB |
| [25.02](25-02.md) | Implement scope=all and scope=first lists |
| [25.03](25-03.md) | Create Authors with All authors/First authors, search/pagination/sorting, and Affiliation |
| [25.04](25-04.md) | Create Affiliations with required Institute and optional Description |
| [25.05](25-05.md) | Allow inline editing/addition/deletion only for Owner/Manager/Admin, with Zod, auditing, and confirmation |
| [25.06](25-06.md) | Block deletion of authors linked to Papers and affiliations linked to authors |

### 26 — Implement Venue Directory

Parent: [26-venues.md](../26-venues.md).

| Local issue | Scope (original step) |
| --- | --- |
| [26.01](26-01.md) | Reuse Venue and the optional Paper reference in the fixed schema |
| [26.02](26-02.md) | Implement list/create/update/delete for unreferenced venues, verifying project and role |
| [26.03](26-03.md) | Create the Full venue name \| Publication year \| Actions table with search, pagination, and sorting |
| [26.04](26-04.md) | Allow Edit mode and new rows for Owner/Manager/Admin |
| [26.05](26-05.md) | Block deletion when a Paper references the venue |

### 27 — Implement Secure Import Storage and Project Job Progress

Parent: [27-import-intake.md](../27-import-intake.md).

| Local issue | Scope (original step) |
| --- | --- |
| [27.01](27-01.md) | Create Start a new import, Import history, and project-scoped details with CSV/ |
| [27.02](27-02.md) | Validate extension/format, size, MIME, and basic content |
| [27.03](27-03.md) | Store originals in private temporary storage using opaque references |
| [27.04](27-04.md) | Implement only Uploading/Queued/Failed/Cancelled |
| [27.05](27-05.md) | Return progress, safe failure, retry eligibility/expiry, and allowed actions |
| [27.06](27-06.md) | Allow cancellation only before parsing/commit |
| [27.07](27-07.md) | Apply an initial 24-hour storage TTL to protect against interrupted workers, without a dedicated cleanup job |

### 28 — Provide one CSV Import page organised into progressive sections.

Parent: [28-csv-import.md](../28-csv-import.md).

| Local issue | Scope (original step) |
| --- | --- |
| [28.01](28-01.md) | Open a Queued batch and inspect its existing temporary file |
| [28.02](28-02.md) | Create a limited preview and mapping: Title required |
| [28.03](28-03.md) | Offer independent optional active Source and Strategy selectors |
| [28.04](28-04.md) | Validate rows and display numbered errors |
| [28.05](28-05.md) | Save persists option/mapping/metadata/choice snapshots and creates one idempotent job |
| [28.06](28-06.md) | The worker reparses/revalidates, generates missing keys using a prefix/atomic counter, and atomically… |
| [28.07](28-07.md) | Persist total/valid/invalid/created/ignored counts and ImportBatchIgnoredRow only for invalid rows… |
| [28.08](28-08.md) | Allow cancellation before commit |

### 29 — Implement BibTeX and EndNote Import Parsers

Parent: [29-bibler-import.md](../29-bibler-import.md).

| Local issue | Scope (original step) |
| --- | --- |
| [29.01](29-01.md) | Confirm BiBler access, documentation, fixtures, and legacy contract |
| [29.02](29-02.md) | The worker reads a Queued batch's temporary file and sends it to the internal service with timeouts,… |
| [29.03](29-03.md) | Validate responses against a fixed normalized-reference contract |
| [29.04](29-04.md) | Persist ordered candidates and safe warnings/errors on the existing batch without creating Papers yet |
| [29.05](29-05.md) | Create a preview and Source/Strategy selection before subsequent review when configured |
| [29.06](29-06.md) | Integrate user-approved asynchronous commit through the shared workflow |
| [29.07](29-07.md) | Do not persist raw responses/parser logs or intermediate debugging artifacts |

### 30 — Implement Screening Assignment and Reviewer Decision Flow

Parent: [30-screening.md](../30-screening.md).

| Local issue | Scope (original step) |
| --- | --- |
| [30.01](30-01.md) | Resolve effective configuration and Open/Closed state on the server |
| [30.02](30-02.md) | Create Dashboard/Papers/Screen/Assignments/Screening decisions navigation and tables with search, filters,… |
| [30.03](30-03.md) | List Papers as All/Included/Excluded/Pending/Under review/In conflict |
| [30.04](30-04.md) | Owner/Manager/Admin previews and assigns to non-Guest members with reviewers per Paper and all/count |
| [30.05](30-05.md) | Use active Papers in the first phase and permitted previous-phase output thereafter |
| [30.06](30-06.md) | Create Screen next pending, configured fields, Source query highlighting, note, and Save and next |
| [30.07](30-07.md) | Only the assignee edits their own decision in an open phase |

### 31 — Implement Screening Conflict Resolution and Validation Workspaces

Parent: [31-screening-conflicts-validation.md](../31-screening-conflicts-validation.md).

| Local issue | Scope (original step) |
| --- | --- |
| [31.01](31-01.md) | Resolve phase overrides or project defaults |
| [31.02](31-02.md) | Add results/assignments/history to conflicting Paper details |
| [31.03](31-03.md) | Allow manual Add reviewer for non-Guests with Normal/Veto/Info and a note |
| [31.04](31-04.md) | Implement Decision/Criteria and Unanimity/Majority rules |
| [31.05](31-05.md) | Create Validation only when enabled: Assign validation, Validate, assignments, and results, with member… |
| [31.06](31-06.md) | Preview uses final Excluded Papers, all_excluded or excluded_by_criteria scope, percentage, and uniform… |
| [31.07](31-07.md) | Keep the validated preview set fixed |
| [31.08](31-08.md) | Reuse Screen for the caller's validation decision with server-resolved Info/Normal/Veto type |
| [31.09](31-09.md) | Persist run/configuration snapshot/idempotency, assignments, decisions, and history |

### 32 — Implement Screening Progress Dashboard and Agreement Statistics

Parent: [32-screening-metrics.md](../32-screening-metrics.md).

| Local issue | Scope (original step) |
| --- | --- |
| [32.01](32-01.md) | Create separate dashboard, Screening/Validation progress, and statistics endpoints for each flow |
| [32.02](32-02.md) | Calculate workload from active assignments |
| [32.03](32-03.md) | Create personal/global cards, participant/Total bars, and authorized actions |
| [32.04](32-04.md) | Derive outcomes, decisions by user, historical conflicts, and criteria percentages |
| [32.05](32-05.md) | Save originalScreeningOutcome when creating a validation assignment and use it for Matched rather than… |
| [32.06](32-06.md) | Calculate Kappa on the server when enabled and at least two comparable reviewers exist |
| [32.07](32-07.md) | Refresh read models after assignments, decisions, cancellations, validation, and outcome changes |

### 33 — Implement Quality Assessment Assignment and Assessment Flow

Parent: [33-qa-assessment.md](../33-qa-assessment.md).

| Local issue | Scope (original step) |
| --- | --- |
| [33.01](33-01.md) | Resolve the existing QA configuration |
| [33.02](33-02.md) | Create an Open/Closed workspace with Dashboard/Papers/Assess/Assignments/Assessed/Results |
| [33.03](33-03.md) | Owner/Manager/Admin opens/closes and creates/cancels assignments |
| [33.04](33-04.md) | Distribute evenly among non-Guest members and persist the run/assignments |
| [33.05](33-05.md) | Implement the caller's next Paper, preview, ordered questions, and Save and next |
| [33.06](33-06.md) | Calculate the score sum on the server and mark completion/outcome |
| [33.07](33-07.md) | Display searchable/sortable/paginated tables |

### 34 — Implement QA Results, Low-quality Exclusion, and Progress

Parent: [34-qa-results.md](../34-qa-results.md).

| Local issue | Scope (original step) |
| --- | --- |
| [34.01](34-01.md) | Reuse assignments/responses/outcomes |
| [34.02](34-02.md) | Create cards and member/Total bars, a Paper key \| Title \| Score \| Assessed table,… |
| [34.03](34-03.md) | Titles open existing assessments, read-only where required |
| [34.04](34-04.md) | Implement a preview of completed, non-excluded Papers below the current cut-off |
| [34.05](34-05.md) | Owner/Manager/Admin confirms that same validated set |
| [34.06](34-06.md) | Do not automatically exclude by score |

### 35 — Implement Classification Assignment and Classification Workspace

Parent: [35-classification.md](../35-classification.md).

| Local issue | Scope (original step) |
| --- | --- |
| [35.01](35-01.md) | Resolve module/configuration/permissions and eligibility: active, non-excluded, and Included after QA when… |
| [35.02](35-02.md) | Create All/Classified/Excluded/Pending Papers and My/My pending/My classified/All assignments with search,… |
| [35.03](35-03.md) | Owner/Manager/Admin previews and distributes evenly among non-Guests |
| [35.04](35-04.md) | Create Reference lists by configured name or label |
| [35.05](35-05.md) | Create next Paper, details, and authorized Add/Edit/Delete Classification |
| [35.06](35-06.md) | Render Simple, List, DynamicList, dependent fields, and subcategories on the first save |
| [35.07](35-07.md) | Persist Classification/ClassificationValue/Assignment/ReferenceList/Value in the fixed schema without… |
| [35.08](35-08.md) | Allow reassignment by assignee/Owner/Manager/Admin to another eligible member: close the current… |

### 36 — Implement the data extraction dashbord and progess

Parent: [36-classification-progress.md](../36-classification-progress.md).

| Local issue | Scope (original step) |
| --- | --- |
| [36.01](36-01.md) | Create Hono dashboard/progress reads calculated on the server per project |
| [36.02](36-02.md) | Personal: active, non-excluded assignments, completed and pending |
| [36.03](36-03.md) | Global: active eligible non-excluded Papers, classified Papers with active records, and pending Papers |
| [36.04](36-04.md) | Create All/Classified/Pending/gauge cards and Completed/Assigned bars by member, plus Total… |
| [36.05](36-05.md) | Reassigned work remains pending for the new assignee until saved/confirmed, according to this task's API rule |
| [36.06](36-06.md) | Create Assign/Classify/Progress actions and Results Coming soon until Reporting integration |

### 37 — Implement Classification Results and Reporting Charts

Parent: [37-reporting-charts.md](../37-reporting-charts.md).

| Local issue | Scope (original step) |
| --- | --- |
| [37.01](37-01.md) | Create an active-classification list with key/title and server-derived category/subcategory columns |
| [37.02](37-02.md) | Implement key/title search, value filters, allowed sorting, and pagination |
| [37.03](37-03.md) | Load only active chart definitions from Settings: Simple by one category and Comparative by two, with… |
| [37.04](37-04.md) | Calculate from active classifications and provide an accessible table using the same data |
| [37.05](37-05.md) | Multi-value charts count each selected value and may exceed total/100% |
| [37.06](37-06.md) | Single-value charts open filtered results |
| [37.07](37-07.md) | Reuse existing definitions/data without snapshots, files, jobs, publication, or chart history |

### 38 — Implement Reporting Authors and Venues

Parent: [38-reporting-authors-venues.md](../38-reporting-authors-venues.md).

| Local issue | Scope (original step) |
| --- | --- |
| [38.01](38-01.md) | Define reportable: active, non-excluded, completed saved classification/extraction |
| [38.02](38-02.md) | Create authorized, project-scoped, read-only queries with server-side search, sorting, and pagination |
| [38.03](38-03.md) | Authors: include only reportable links and distinguish Included Papers from First-author Papers using… |
| [38.04](38-04.md) | Venues: group reportable Papers by venue and year |
| [38.05](38-05.md) | Create tables without Add/Edit/Delete |

### 39 — Implement CSV and BibTeX Reporting Exports

Parent: [39-csv-bibtex-exports.md](../39-csv-bibtex-exports.md).

| Local issue | Scope (original step) |
| --- | --- |
| [39.01](39-01.md) | Define operations by export type, authorized within the current project |
| [39.02](39-02.md) | Create cards for Papers CSV |
| [39.03](39-03.md) | Resolve scope on the server: active Papers |
| [39.04](39-04.md) | CSV includes the specified fixed fields: key/title/DOI/URL/preview/abstract/year and, depending on the… |
| [39.05](39-05.md) | Generate UTF-8 with separator/quote/newline escaping |
| [39.06](39-06.md) | Audit type, actor, timestamp, and count |

### 40 — Implement Python and R Statistical Analysis Exports

Parent: [40-python-r-exports.md](../40-python-r-exports.md).

| Local issue | Scope (original step) |
| --- | --- |
| [40.01](40-01.md) | Restrict generation to Owner/Manager/Admin and validate categories against the active project configuration |
| [40.02](40-02.md) | Allow Nominal/Continuous/Text per category |
| [40.03](40-03.md) | Build a reportable snapshot with key/title/authors/year/venue/Source/Strategy and categories |
| [40.04](40-04.md) | Python ZIP: CSV, relis_statistics_kernel |
| [40.05](40-05.md) | Generate appropriate analysis families: frequency/bar, descriptive/box/violin, evolution, and… |
| [40.06](40-06.md) | Stream the ZIP directly and record ReportingExportAudit with actor/language/type… |
| [40.07](40-07.md) | Treat labels and values as data when generating code |

### 41 — Implement backup/restore and hardening

Parent: [41-backup-hardening.md](../41-backup-hardening.md).

| Local issue | Scope (original step) |
| --- | --- |
| [41.01](41-01.md) | Define the Control DB catalog, encrypted-object metadata/checksum/key version/expiry, restore plans/runs,… |
| [41.02](41-02.md) | Implement requestProjectBackup, getBackupStatus, listProjectBackups, and downloadBackupArtifact using jobs… |
| [41.03](41-03.md) | Implement validateRestorePlan, requestProjectRestore, and verifyRestoredProject: target environment,… |
| [41.04](41-04.md) | In production, create a safety backup and perform post-restore verification |
| [41.05](41-05.md) | Create Backups, a restore wizard, and reusable high-impact confirmation |
| [41.06](41-06.md) | Implement configureRetentionPolicy, previewRetentionAction, and executeRetentionAction with versioned… |
| [41.07](41-07.md) | Send optional localized notifications without secrets or artifact access links |

### 42 — Complete CI/CD and production deployment specification

Parent: [42-cicd-deployment.md](../42-cicd-deployment.md).

| Local issue | Scope (original step) |
| --- | --- |
| [42.01](42-01.md) | Create baseline CI early using actual scripts: lint, typecheck, tests, Prisma migration checks, parity,… |
| [42.02](42-02.md) | Document web/API/worker, Control/Project PostgreSQL, object storage, proxy, mail, and backups |
| [42.03](42-03.md) | Externalize secrets and document rotation without leakage into logs/artifacts |
| [42.04](42-04.md) | Define a release runbook with Control/Project migrations, template publication, compatibility, job… |
| [42.05](42-05.md) | Document backup, rollback, incident recovery, and isolation verification without exposing data |
| [42.06](42-06.md) | Rehearse the runbook in a test environment and record results |
