# 40 — Implement Python and R Statistical Analysis Exports

## Source and usage

- Source: [Implement Python and R Statistical Analysis Exports](https://app.notion.com/p/Implement-Python-and-R-Statistical-Analysis-Exports-3d04993602938067bb0fddbe3c03e9b9).
- Notion ID: `3d04993602938067bb0fddbe3c03e9b9`.
- Source domain: Reporting & Export.
- Source status on 2026-09-20: Backlog; this does not represent local progress.
- Type: this file reproduces the full Notion card content (all 15 properties, fetched 2026-09-28), plus a locally authored completion checklist.
- Source `Blocked by` relation: Empty.

First read the [README](README.md), [pending decisions](PENDING-DECISIONS.md), [stack](../stack.yml), and [required structure](../project-structure.md). Confirm the target repository and existing implementation before editing code. The structure is the required target, not evidence that any feature has been implemented.

## Full Notion card content (fetched 2026-09-28)

- **Status:** Backlog
- **Description:** Provide downloadable Python and R analysis environments for a project's reportable Classification/Data Extraction results.
- **Functional Domain:** Reporting & Export
- **Behavior:** The new Relis keeps the legacy principle: the user selects the analytical type of each extraction category, then downloads a ZIP containing the current result dataset and generated analysis scripts. The server generates the archive on demand and sends it directly to the browser. Generated archives and CSV files are not retained on the server.
- **Users and permissions:** Only the Project Owner, Project Managers, and Application Administrators can generate an export.
- **UI behavior:**
  - Within Reporting > Exports, provide two sections: Python analysis environment; R analysis environment.
  - **Export configuration** — before generation, display active Classification/Data Extraction categories in a table: Category | Export data type. For each category, allow: Nominal; Continuous; Text. Preselect a suitable value from the category configuration; allow the user to change it before generation. Text fields remain in the CSV but are omitted from generated statistical analysis calls.
  - The exported CSV also contains fixed Paper metadata fields: Paper key; title; authors; publication year; venue; Paper Source; Search Strategy. Multi-value categories are exported as one CSV value separated by `|`. Subcategory columns use an unambiguous parent-child label.
  - **Python export** — "Generate Python export" downloads one ZIP containing: the current reportable dataset as CSV; `relis_statistics_kernel.py`; `relis_statistics_playground.py`; `requirements.txt`. The generated Python environment provides legacy analysis families appropriate to the selected field types: frequency tables and bar plots for nominal fields; descriptive statistics, box plots, and violin plots for continuous fields; evolution plots; comparative tables, charts, chi-squared tests, and correlation tests where applicable.
  - **R export** — "Generate R export" downloads one ZIP containing: the current reportable dataset as CSV; an R analysis library script; an R playground/configuration script. The generated R playground contains ready-to-uncomment calls for the same descriptive, evolutive, and comparative analyses supported by the legacy export.
- **API behavior:**
  - Provide authorised operations to generate a Python or R export. For each generation: validate that submitted category identifiers belong to the active project Classification/Data Extraction configuration; build a snapshot from current reportable Papers; flatten generic category values, dependencies, and subcategories into stable CSV columns; generate the language-specific scripts from the selected data types; stream one ZIP response to the browser; do not execute Python or R on the server; do not persist the generated ZIP or CSV; record an export audit event containing actor, language, generation time, category-type configuration, and exported Paper count.
- **Data model:** Add `ReportingExportAudit` in the Project DB: actor; language (Python or R); category-type configuration snapshot; reportable Paper count; created time; generation result. No generated-file storage is required.
- **Acceptance criteria:**
  - [ ] Only authorised users can generate Python or R exports.
  - [ ] Each archive contains the current reportable dataset only.
  - [ ] Each active extraction category can be marked Nominal, Continuous, or Text.
  - [ ] The Python ZIP contains CSV, kernel, playground, and requirements files.
  - [ ] The R ZIP contains CSV, library, and playground/configuration scripts.
  - [ ] Multi-value categories and subcategories export consistently.
  - [ ] Python and R are never executed on the server.
  - [ ] Generated archives are not retained after download.
  - [ ] Every generation is auditable.
- **Assigned to:** Empty
- **Severity:** Empty
- **Blocked by:** Empty
- **Jobs behavior:** Empty
- **Audit:** Empty

## Objective and summarized requirements

Download Python/R analysis environments with a reportable snapshot without running either language on the server.

## Proposed prerequisites

- [35 — Implement Classification Assignment and Classification Workspace](35-classification.md)
- [38 — Implement Reporting Authors and Venues](38-reporting-authors-venues.md)
- [39 — Implement CSV and BibTeX Reporting Exports](39-csv-bibtex-exports.md)

The dependencies above are implementation inferences. The original Notion relation is preserved separately; any cycles have not been removed.

## Step-by-step instructions for the LLM

1. Restrict generation to Owner/Manager/Admin and validate categories against the active project configuration.
2. Allow Nominal/Continuous/Text per category; retain Text in CSV but exclude it from statistical calls.
3. Build a reportable snapshot with key/title/authors/year/venue/Source/Strategy and categories; separate multi-values with `|` and use unambiguous parent-child subcategory labels.
4. Python ZIP: CSV, `relis_statistics_kernel.py`, `relis_statistics_playground.py`, and `requirements.txt`. R ZIP: CSV, library, and playground/configuration.
5. Generate appropriate analysis families: frequency/bar, descriptive/box/violin, evolution, and comparative/chi-squared/correlation where applicable; R scripts include calls ready to uncomment.
6. Stream the ZIP directly and record `ReportingExportAudit` with actor/language/type configuration/count/timestamp/result.
7. Treat labels and values as data when generating code; do not execute Python/R or persist ZIP/CSV on the server.

## Definition of Done — completion checklist

Notion's Acceptance criteria (reproduced above) is the primary source; the items below restate it as concrete, testable points and add detail grounded in the API behavior/UI behavior fields.

- [ ] Only the Project Owner, Project Managers, and Application Administrators can trigger a Python or R export — every other role gets a 403.
- [ ] Submitted category identifiers are validated against the project's **active** Classification/Data Extraction configuration server-side — a stale or foreign category ID is rejected before any snapshot is built.
- [ ] The dataset snapshot uses the same "reportable" Paper definition established in [38](38-reporting-authors-venues.md)/[37](37-reporting-charts.md) — only current reportable Papers are included, computed fresh at generation time (no cached/stale snapshot).
- [ ] Each active extraction category can be independently marked Nominal, Continuous, or Text, preselected sensibly from its configuration but user-overridable before generation; Text-typed fields remain present in the CSV but are excluded from every generated statistical call in both the Python and R scripts.
- [ ] The CSV always includes the fixed Paper metadata columns (key, title, authors, publication year, venue, Paper Source, Search Strategy) in addition to the selected category columns.
- [ ] Multi-value category values are serialized as one CSV cell with `|` as the separator; subcategory columns use an unambiguous parent-child label (e.g., `Category>Subcategory`) so they can never be confused with an unrelated top-level category of the same name.
- [ ] The Python ZIP contains exactly: the CSV, `relis_statistics_kernel.py`, `relis_statistics_playground.py`, and `requirements.txt`; the R ZIP contains exactly: the CSV, an R analysis library script, and an R playground/configuration script.
- [ ] Generated scripts implement the field-type-appropriate analysis families described (frequency/bar for nominal; descriptive/box/violin for continuous; evolution; comparative/chi-squared/correlation where applicable in Python; the equivalent set as ready-to-uncomment calls in R) — using the actual selected field types from this generation, not a fixed hard-coded set.
- [ ] Category/field labels and values are treated strictly as **data** when generating the scripts — they are never interpolated in a way that could execute as code (e.g., escape/quote any label used inside a generated Python/R string literal).
- [ ] Neither Python nor R is executed on the server at any point in this flow — script generation is pure string/template assembly.
- [ ] Neither the ZIP nor the CSV is persisted anywhere — the response streams directly to the browser and nothing remains after the request completes.
- [ ] Every generation writes a `ReportingExportAudit` record (actor, language, category-type configuration snapshot, reportable Paper count, created time, generation result).
- [ ] Automated tests cover: rejection of a non-authorised role; rejection of an invalid/foreign category ID; correct CSV shape including multi-value `|`-joining and parent-child subcategory labels; correct file sets in both ZIPs; and a label-injection attempt safely escaped in a generated script.
- [ ] Report which commands were run and their actual results, and explicitly flag that the generated scripts' statistical correctness could not be verified against the legacy tool without the approved fixtures/templates referenced in Gaps.

## Gaps and pending decisions

The source provides no templates or detailed statistical-test applicability rules for the Python/R scripts. Retrieve legacy fixtures/templates (per [pending decisions](PENDING-DECISIONS.md) item 16) and validate generated files in a test environment before claiming parity with the legacy exports — do not invent the statistical templates from this description alone.

## Expected implementation deliverable

Implementation of the described scope, verification evidence, updated documentation, and a report of limitations. This file only guides execution; creating it does not implement the feature or update its Notion status.

## Detailed local sub-issues

Each original step now has a separate draft sub-issue with scope, decision gates, acceptance criteria, and specific verification scenarios. These are local proposals, not new Notion cards. Read [the execution rules](sub-issues/README.md) and implement only the explicitly selected scope.

- [40.01 — Restrict generation to Owner/Manager/Admin and validate categories against the active project configuration](sub-issues/40-01.md)
- [40.02 — Allow Nominal/Continuous/Text per category](sub-issues/40-02.md)
- [40.03 — Build a reportable snapshot with key/title/authors/year/venue/Source/Strategy and categories](sub-issues/40-03.md)
- [40.04 — Python ZIP: CSV, relis_statistics_kernel](sub-issues/40-04.md)
- [40.05 — Generate appropriate analysis families: frequency/bar, descriptive/box/violin, evolution, and…](sub-issues/40-05.md)
- [40.06 — Stream the ZIP directly and record ReportingExportAudit with actor/language/type…](sub-issues/40-06.md)
- [40.07 — Treat labels and values as data when generating code](sub-issues/40-07.md)
