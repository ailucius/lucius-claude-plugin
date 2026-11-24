---
name: compliance-matrix-excel
description: Build a tender compliance matrix as an Excel workbook (.xlsx) from a Lucius pack analysis, one row per requirement with the quote in its original language, an English line, document, page and severity, and blank owner and status columns for the bid team. Use when the user asks for a compliance matrix, requirements matrix, compliance checklist or requirements spreadsheet for a tender, or asks to export the requirements to Excel.
---

# Compliance matrix in Excel

Builds an .xlsx from the requirements of a finished Lucius analysis. Every cell comes from the Lucius tools or is left blank for the team. Never fill in an owner, a status, an answer or evidence.

## 1. Get the analysis

You need an analysis id:

- from earlier in the conversation (`analysis_id` from `analyze_pack`, or `analysis.analysis_id` from `get_tender`);
- from a Lucius app link whose path ends in `/dashboard/` and the analysis id;
- otherwise follow the tender-check skill, steps 1 to 3, to find the tender and run the analysis.

## 2. Read every requirement

Call `get_pack_analysis` with `id`, `requirements_offset` 0, `requirements_limit` 1000 and `include_deliverables` false.

- `status` must be "complete". If it is "queued" or "processing", wait and check again; if "failed", give the `reason`.
- While `result.requirements_next_offset` is not null, call again with `requirements_offset` set to it, and add the rows.
- Keep the rows in `position` order.

## 3. Check what the workspace's plan includes

When `result.requirements_locked` is false, the matrix holds every requirement the tool returned, not only the deciding ones, and nothing is labelled "deciding requirements only".

If `result.requirements_locked` is true, the result holds only the requirements that decide the bid. Build the workbook from those rows anyway and label it clearly:

- file name ending "deciding requirements only";
- a first line on the Matrix sheet: "Deciding requirements only. The full list is not included for this tender on this workspace's plan.";
- the About sheet gives `result.locked_reason` exactly as returned and the link (see step 5).

## 4. Build the workbook

**Sheet "Matrix"**, one row per requirement, with these columns:

| Column | Field |
| --- | --- |
| # | `position` |
| Requirement | `requirement_plain` |
| Quote (original language) | `original_text` |
| English (Lucius) | `requirement_en`, blank when null |
| Clause or reference | `requirement` |
| Section | `section` |
| Document | `document` |
| Page | `page` |
| Page approximate | "yes" when `page_is_approximate` is true |
| Quote checked | "yes" or "no" from `quote_verified`, blank when null |
| Severity | `severity` |
| Why it matters | `why` |
| Deciding | "yes" when `deciding` is true |
| Owner | blank |
| Status | blank |
| Evidence and notes | blank |

Header row bold and frozen, a filter on the header row, text wrapped in the Requirement, Quote and Why columns, and wide enough columns to read. Do not translate, shorten or correct quotes.

**Sheet "Forms"**, only when `result.forms_status` is "ready" and `result.forms` is not empty: `name`, `name_en`, `document`, `page`, `mandatory`, `purpose`, and from `named_in` the requirement `position`, its `page` and its `quote`. When `result.forms_status` is anything else, put `result.forms_note` on the About sheet instead.

**Sheet "About"**:

- `result.title`, `result.buyer`, `result.deadline`;
- the analysis `app_url`;
- `result.requirements_total` and the number of rows in the Matrix sheet;
- if `result.requirements_final` is false: "A second read was still running; more requirements may be added. Read the analysis again later.";
- if `result.coverage.complete` is false: each of `result.coverage.notes`;
- when `result.notice` is present: its `source_url` and its `attribution`, which must stay with the data;
- the date you read the analysis.

## 5. Finish

Give the file, then in a few lines: the rows as "N of M rows", where N is the number of rows in the Matrix sheet and M is `result.requirements_total`; how many deciding; and any coverage gaps.

If `result.requirements_locked` is true, end with `result.locked_reason` exactly as returned. If it contains a link, end with that link. If not, end with the analysis `app_url`, where the full list can be opened in the Lucius app. Never write a different link and never quote a price.

## Errors

Errors come as JSON with `error` and `status`. On 402 the workspace has used its Lucius allowance: say the request did not run, repeat the `error` text and do not retry. For other errors, repeat the `error` text.

## Surface notes

- Claude: create the .xlsx with Claude's built-in Excel file creation. In Claude Code without it, write the file with a spreadsheet library such as openpyxl if it is installed, and tell the user the file path.
