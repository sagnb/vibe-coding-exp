# 37 — Implement Classification Results and Reporting Charts

## Source and usage

- Source: [Implement Classification Results and Reporting Charts](https://app.notion.com/p/Implement-Classification-Results-and-Reporting-Charts-3b349936029380e7b9eceeaec3bd7a1a).
- Notion ID: `3b349936029380e7b9eceeaec3bd7a1a`.
- Source domain: Reporting & Export.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide a read-only Reporting dashboard and Classification/Data Extraction charts for the current project. The new relis reproduces the useful reporting information from the legacy: Classification workload counts and distributions for compatible extracted categories.
- **Functional Domain:** Reporting & Export
- **Behavior:** Empty
- **Users and permissions:**
  - Every active project member, including Guests, can view Reporting dashboard and charts for the current project.
  - An Application Administrator can view them within their administrative project scope.
- **UI behavior:**
  - Within Reporting, provide: Classification result; Charts.
  - **Classification results** — a searchable, sortable, paginated table of active saved Classifications: Paper key | Title | [configured Data Extraction category and subcategory columns]. One row per Paper with an active saved Classification. Category columns/labels are derived server-side from the project's active Data Extraction configuration. Each cell shows the saved value for that Paper/category. Supports text search by Paper key/title, and sorting by Paper fields and compatible category columns. A Paper without a saved Classification does not appear here — pending/assignment/exclusion views remain part of the Classification workspace. Title opens the existing Paper/Classification detail view.
  - **Charts** — display only the active Custom Chart Reports configured in Settings > Reporting ([21](21-project-settings.md)). A chart definition contains: title; report type (Simple or Comparative); value category; reference category when Comparative; one or more chart types (Pie, Bar, or Line).
    - Simple chart — groups saved Classification records by one configured category. Displays the configured title, every configured chart type, and an accessible table: Value | Papers | Percentage.
    - Comparative chart — groups one configured value category against a configured reference category. Displays the configured title, every configured chart type, and an accessible comparison table derived from the same chart data.
    - Multi-value categories — count every selected value; allow totals/percentages to exceed the number of classified Papers; do not provide click-through filtering for a single selected value. For single-value categories, clicking a chart segment or table row opens the existing Classification/Data Extraction result list filtered by that value.
    - Invalid configuration — when a configured category has been removed, disabled, or is no longer usable by the report definition: do not render a misleading chart; display a clear configuration warning; link authorised users to Settings > Reporting. When no active chart is configured, display an empty state.
- **API behavior:**
  - Provide read operations to: list saved Classification/Data Extraction results; retrieve active reporting chart definitions with their calculated data.
  - The Classification results operation supports: search by Paper key and Title; category-value filters; allowed sorting; pagination.
  - Return: Paper key and Title; the active Data Extraction category definitions required as table columns; the saved category values for each active Classification.
  - Rules: return only Papers with an active saved Classification; resolve result columns and chart definitions from the current project configuration; calculate chart data only from active saved Classification records; a chart selection reuses the Classification results operation with the corresponding category-value filter; do not accept category definitions or chart configuration from the browser; do not create dashboard counts, snapshots, generated files, publication records, or chart-history records.
- **Data model:** Reuse the Reporting chart definitions configured in Project Settings. Each chart definition must store: title; report type (Simple/Comparative); value category reference; optional reference category reference; selected chart types; active state. No additional Reporting runtime tables are required.
- **Acceptance criteria:**
  - [ ] Every active project member can view project-isolated dashboard counts and charts.
  - [ ] Dashboard displays All, Processed, Pending, Assigned to me, and Excluded counts.
  - [ ] Only active chart definitions from Project Settings are displayed.
  - [ ] Simple reports display configured Pie, Bar, and/or Line charts with an accessible value table.
  - [ ] Comparative reports group configured values against their configured reference category.
  - [ ] Multi-value charts count every selected value and do not expose invalid click-through filtering.
  - [ ] Single-value chart selections open the corresponding filtered Classification/Data Extraction results.
  - [ ] Invalid chart configuration produces a clear warning rather than an incorrect chart.
  - [ ] No snapshots, exports, public reporting, workers, or report-publication flow is implemented.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Display project-scoped classification results and configured charts in read-only mode.

## Proposed prerequisites

- [21 — Implement Direct Project Settings Management](21-project-settings.md)
- [35 — Implement Classification Assignment and Classification Workspace](35-classification.md)
- [36 — Implement the data extraction dashbord and progess](36-classification-progress.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Create an active-classification list with key/title and server-derived category/subcategory columns.
2. Implement key/title search, value filters, allowed sorting, and pagination; titles reuse details.
3. Load only active chart definitions from Settings: Simple by one category and Comparative by two, with selected Pie/Bar/Line types.
4. Calculate from active classifications and provide an accessible table using the same data.
5. Multi-value charts count each selected value and may exceed total/100%; do not offer per-value click-through in this case.
6. Single-value charts open filtered results; invalid categories produce warnings and Settings links for authorized users.
7. Reuse existing definitions/data without snapshots, files, jobs, publication, or chart history.

## Definition of Done — completion checklist

This card's Acceptance criteria (reproduced above) directly contradicts its own API behavior on dashboard counts (see Gaps); the checklist below implements the uncontested results-table and charts core and explicitly withholds the conflicting dashboard-counts requirement pending a decision.

- [ ] The Classification results table includes only Papers with an active saved Classification, shows Paper key/Title plus category/subcategory columns derived server-side from the project's active Data Extraction configuration, and supports search (key/title), value filters, sorting, and pagination.
- [ ] Clicking a title opens the existing Paper/Classification detail view — this page adds no separate edit path.
- [ ] Only **active** Custom Chart Reports configured in [21 — Project Settings](21-project-settings.md) are rendered; a disabled or deleted chart definition never appears.
- [ ] A Simple chart groups saved Classifications by its one configured category and renders every configured chart type (Pie/Bar/Line) plus an accessible Value | Papers | Percentage table built from the **same** underlying data as the visual chart.
- [ ] A Comparative chart groups its value category against its reference category and renders every configured chart type plus an accessible comparison table from the same data.
- [ ] For a multi-value category, every selected value is counted (totals/percentages may exceed the classified-Paper count) and no per-value click-through filtering is offered; for a single-value category, clicking a segment/row opens the Classification/Data Extraction results filtered by that value.
- [ ] When a chart's configured category has been removed/disabled/is otherwise unusable, the UI shows a clear configuration warning (never a silently wrong or empty-looking chart) with a link to Settings > Reporting for authorised users; when no chart is configured at all, an empty state is shown instead.
- [ ] Category definitions and chart configuration are never accepted from the browser — both are resolved entirely server-side from the current project's stored configuration.
- [ ] No snapshot, generated file, publication record, chart-history record, or background job is created by this feature — chart data is computed live, on read, from current saved Classification records.
- [ ] Automated tests cover: the results table excluding Papers without an active Classification; a Simple and a Comparative chart producing matching visual/table numbers; multi-value counting exceeding 100%; and the invalid-configuration warning path.
- [ ] Report which commands were run and their actual results, and explicitly record whether the dashboard-counts requirement (All/Processed/Pending/Assigned to me/Excluded) was implemented or deferred pending the decision in Gaps — do not silently pick one side.

## Gaps and pending decisions

Acceptance criteria require All/Processed/Pending/Assigned to me/Excluded dashboard counts, but the API behavior explicitly says "do not create dashboard counts" — a direct contradiction on this card. Settings/Wizard ([18](18-protocol-wizard.md)/[21](21-project-settings.md)) allow a Bubble chart type for Compare reports, while this page's UI/API behavior allow only Pie/Bar/Line. Do not mark full acceptance without resolving both — see [pending decisions](PENDING-DECISIONS.md) item 13.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [37.01 — Create an active-classification list with key/title and server-derived category/subcategory columns](sub-issues/37-01.md)
- [37.02 — Implement key/title search, value filters, allowed sorting, and pagination](sub-issues/37-02.md)
- [37.03 — Load only active chart definitions from Settings: Simple by one category and Comparative by two, with…](sub-issues/37-03.md)
- [37.04 — Calculate from active classifications and provide an accessible table using the same data](sub-issues/37-04.md)
- [37.05 — Multi-value charts count each selected value and may exceed total/100%](sub-issues/37-05.md)
- [37.06 — Single-value charts open filtered results](sub-issues/37-06.md)
- [37.07 — Reuse existing definitions/data without snapshots, files, jobs, publication, or chart history](sub-issues/37-07.md)
