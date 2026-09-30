# 32 — Implement Screening Progress Dashboard and Agreement Statistics

## Source and usage

- Source: [Implement Screening Progress Dashboard and Agreement Statistics](https://app.notion.com/p/Implement-Screening-Progress-Dashboard-and-Agreement-Statistics-3b349936029380639b34d5798e14cfd8).
- Notion ID: `3b349936029380639b34d5798e14cfd8`.
- Source domain: Screening.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide read-only progress and statistics pages for each screening phase. Metrics are calculated from existing screening assignments, decisions, validation assignments, validation decisions, and derived Paper outcomes. This ticket does not create assignments, screening decisions, conflicts, or validation decisions.
- **Functional Domain:** Screening
- **Behavior:** Legacy ReLiS provides personal/overall screening progress, user completion, result counts, gauges, conflict counts, validation statistics, and optional Kappa calculation/interpretation. NeoReLiS keeps these measures but calculates them from canonical assignment, decision, validation, and resolution records.
- **Users and permissions:** All project members, including Guests, can view dashboard, progress, and statistics in read-only mode. Project Owners, Project Managers, and authorised Application Administrators retain access to their existing assignment actions.
- **UI behavior:**
  - Two action groups for the current screening phase — Screening: Assign papers, Screen, Progress, Statistics. Validation (display only when enabled): Assign validation, Validate, Progress, Statistics.
  - Progress cards for the current phase: My screening progress (All assignments, Screened, Pending, In conflict, completion gauge); Overall screening assignment progress (same fields); My validation progress (All validation assignments, Validated, Pending, completion gauge); Overall validation progress (same fields). For screening, the gauge represents completed assignments that are **not** currently in conflict. Hide the Validation action group/cards when validation is disabled.
  - **Progress pages** — Screening → Progress opens a dedicated Screening progress page; Validation → Progress opens a dedicated Validation progress page. Each shows one row per assigned participant: User | Completed / Assigned | Completion, with a progress bar per user and a Total row.
  - **Statistics pages** — **Screening Statistics**: Results (Included, Excluded, In conflict, In review, Pending, Total); Agreement (Kappa result and interpretation only when enabled — show an unavailable state when fewer than two comparable reviewers exist); Decisions by user (User | Included | Excluded | In conflict); Conflict status (Resolved as included | Resolved as excluded | Currently in conflict); Exclusion criteria statistics (Criterion | Papers | Percentage); Inclusion criteria statistics when configured.
  - **Validation Statistics**: a validation-results table (Paper | Validation decision | Matched); a general validation score (Matches / Total validations | Percentage); validation score by user (Validator | Matches / Validations | Percentage). "Matched" indicates whether the validation decision matches the Paper's screening outcome.
- **API behavior:**
  - Provide a read-only phase-dashboard endpoint containing enabled action links, progress cards, and permissions. Provide separate read-only endpoints for: screening progress; validation progress; screening statistics; validation statistics. Scope every result to the current project and screening phase.
  - Calculate: screening workload from active screening assignments; validation workload from active validation assignments; screening gauge completion as completed assignments excluding currently conflicted Papers; validation gauge completion as completed validation assignments; outcome statistics from derived phase-Paper outcomes; conflict-status statistics from outcome-history events; criteria percentages from recorded criterion selections; validation matching against the original screening outcome saved when the validation assignment was created.
  - Calculate Kappa server-side only when enabled and when at least two comparable reviewer decisions exist. Refresh these read models after assignment, decision, validation, cancellation, or outcome changes.
- **Data model:** (stored in the Project DB)
  - `ScreeningStatisticsSnapshot` — phase; calculated time; dashboard, progress, screening-statistics, and validation-statistics values.
  - `ScreeningOutcomeHistory` — phase, Paper, previous outcome, new outcome, operation, actor, time. Used for conflict-status statistics.
  - `ScreeningValidationAssignment` — add `originalScreeningOutcome` snapshot. Used to calculate Matched independently of later Normal or Veto changes.
- **Acceptance criteria:**
  - [ ] Screening and validation progress are calculated separately.
  - [ ] Dashboard cards and Progress pages show correct completed, pending, conflict, and total counts.
  - [ ] Screening Statistics show outcomes, user decisions, conflicts, criteria, and Kappa when enabled.
  - [ ] Validation Statistics show validation decisions, matches, global score, and per-validator score.
  - [ ] Matched uses the saved original screening outcome.
  - [ ] Metrics refresh after relevant workflow changes and remain project-isolated.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Display read-only phase metrics that distinguish Screening from Validation.

## Proposed prerequisites

- [30 — Implement Screening Assignment and Reviewer Decision Flow](30-screening.md)
- [31 — Implement Screening Conflict Resolution and Validation Workspaces](31-screening-conflicts-validation.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Create separate dashboard, Screening/Validation progress, and statistics endpoints for each flow.
2. Calculate workload from active assignments; the Screening gauge counts completed assignments outside conflicts, while Validation counts completed validations.
3. Create personal/global cards, participant/Total bars, and authorized actions; hide disabled Validation.
4. Derive outcomes, decisions by user, historical conflicts, and criteria percentages; record `ScreeningOutcomeHistory` and snapshots as specified.
5. Save `originalScreeningOutcome` when creating a validation assignment and use it for Matched rather than comparing with an outcome already changed by Veto/Normal.
6. Calculate Kappa on the server when enabled and at least two comparable reviewers exist; show unavailable when insufficient.
7. Refresh read models after assignments, decisions, cancellations, validation, and outcome changes.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/Data model fields.

- [ ] This feature is strictly read-only: it creates no assignment, screening decision, conflict, or validation decision — it only aggregates records produced by [30](30-screening.md) and [31](31-screening-conflicts-validation.md).
- [ ] Screening and Validation progress/statistics are computed and displayed as fully separate metrics — a screening completion count never folds in validation activity or vice versa.
- [ ] The screening completion gauge counts completed assignments **excluding** Papers currently `In conflict`; the validation gauge counts completed validation assignments.
- [ ] Every progress/statistics endpoint scopes strictly to the current project and phase — cross-phase or cross-project aggregation never happens, even accidentally via a shared cache key.
- [ ] Screening Statistics show: outcome counts (Included/Excluded/In conflict/In review/Pending/Total); decisions-by-user; conflict-status breakdown (resolved-as-included / resolved-as-excluded / currently-in-conflict) sourced from `ScreeningOutcomeHistory`; exclusion (and, when configured, inclusion) criteria percentages; and Kappa + interpretation only when enabled, with an explicit "unavailable" state (never a fabricated 0 or 1) when fewer than two comparable reviewers exist.
- [ ] Validation Statistics show the per-Paper validation-decision/Matched table, the global Matches/Total score, and the per-validator score — "Matched" is computed against `ScreeningValidationAssignment.originalScreeningOutcome`, the outcome snapshot taken **when the validation assignment was created**, not the Paper's possibly-since-changed current outcome (e.g., after a later Veto/Normal validation decision moved it).
- [ ] The Validation action group and its cards/pages are hidden entirely when Screening Validation is disabled for the project/phase.
- [ ] All project members, including Guests, can view every dashboard/progress/statistics page read-only; only Owner/Manager/authorised Admin retain the underlying assignment actions (Assign papers, Assign validation) linked from the dashboard.
- [ ] Read models (e.g., `ScreeningStatisticsSnapshot`) are refreshed after every assignment creation/cancellation, decision submission/edit/revocation, validation decision, and outcome change — a stale snapshot is never served after such an event.
- [ ] Automated tests cover: the screening gauge correctly excluding in-conflict Papers; Kappa's "unavailable" state with fewer than two reviewers; Matched correctly using the original (not current) screening outcome after a simulated later change; and project/phase isolation between two seeded projects.
- [ ] Report which commands were run and their actual results, and record the exact Kappa variant, missing-data handling, and criteria-percentage denominator chosen, since none of these are specified by the source (see Gaps).

## Gaps and pending decisions

The source does not define the Kappa variant (e.g., Cohen's vs. Fleiss'), missing-data handling, or criteria-percentage denominators. Define and document these explicitly before validating statistics — do not assume a formula; see [pending decisions](PENDING-DECISIONS.md) item 16.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [32.01 — Create separate dashboard, Screening/Validation progress, and statistics endpoints for each flow](sub-issues/32-01.md)
- [32.02 — Calculate workload from active assignments](sub-issues/32-02.md)
- [32.03 — Create personal/global cards, participant/Total bars, and authorized actions](sub-issues/32-03.md)
- [32.04 — Derive outcomes, decisions by user, historical conflicts, and criteria percentages](sub-issues/32-04.md)
- [32.05 — Save originalScreeningOutcome when creating a validation assignment and use it for Matched rather than…](sub-issues/32-05.md)
- [32.06 — Calculate Kappa on the server when enabled and at least two comparable reviewers exist](sub-issues/32-06.md)
- [32.07 — Refresh read models after assignments, decisions, cancellations, validation, and outcome changes](sub-issues/32-07.md)
