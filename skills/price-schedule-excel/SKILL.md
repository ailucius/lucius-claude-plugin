---
name: price-schedule-excel
description: Turn the price schedule or bill of quantities that Lucius listed from a tender pack into an Excel workbook (.xlsx) with the buyer's lines, quantities, units and pages, and empty price columns for the bid team. Says plainly when the pack has no price schedule. Use when the user asks for the price schedule, pricing sheet, bill of quantities, BOQ, price form or lines to price for a tender.
---

# Price schedule in Excel

Builds an .xlsx from the price schedule Lucius stored for an analysis. Lucius lists the buyer's lines, quantities, units and pages; it holds no prices. This skill never writes a price, a rate, a discount or a total amount, and never estimates a quantity.

## 1. Get the analysis

You need an analysis id: from earlier in the conversation, from a Lucius app link ending in `/dashboard/` and the analysis id, or by following the tender-check skill, steps 1 to 3.

## 2. Read the price schedule

Call `get_pack_analysis` with `id`, `requirements_offset` 0, `requirements_limit` 1 and `include_deliverables` true. The price schedule comes only with the first page.

`status` must be "complete". Then read `result.price_schedule.status`:

- **"ready"**: build the workbook from `result.price_schedule.schedule` (step 3).
- **"none"**: say plainly that Lucius read this pack and found no price schedule or bill of quantities. If `result.price_schedule.reason` is "ungrounded", say instead that Lucius could not check what it read against the pack, which is not the same as the pack having no price form, and that the team should check the pack. Build no workbook.
- **"pending"**: Lucius is still building it. Read again in a minute.
- **"failed"**: give `result.price_schedule.error` when present, and the analysis `app_url`.
- **"stale"**: it was built from an earlier requirements list. Say so; you may build from it with that label on the About sheet.
- **null**: not built yet. It is built in the Lucius app; give the analysis `app_url`.
- **"locked"**: this workspace's plan does not include the price schedule for this tender. Build no schedule. You may offer the requirements that decide the bid as a compliance matrix instead (the compliance-matrix-excel skill), labelled as the deciding requirements only. Give `result.price_schedule.note` exactly as returned. If it contains a link, end with that link. If not, end with the analysis `app_url`, where the price schedule can be opened in the Lucius app. Never write a different link and never quote a price.

## 3. Build the workbook

From `schedule`:

**Sheet "Price schedule"**, one row per item of `lines`, in order:

| Column | Field |
| --- | --- |
| Ref | `ref` |
| Group | `group_original` |
| Group (English) | `group_en` |
| Description | `description_original` |
| Description (English) | `description_en` |
| Unit | `unit_original` |
| Unit (English) | `unit_en` |
| Quantity | `quantity`, as text exactly as given |
| Includes | `includes_original`, with `includes_en` |
| Specification | `spec_original`, with `spec_en` and `spec_page` |
| Site | `site_original`, with `site_en` |
| Page | `page` |
| Unit price | blank; head it with `unit_price_heading.original` when present |
| Total | blank; head it with `total_heading.original` when present |
| Notes | blank |

You may put a formula in Total (quantity times unit price) only on rows where `quantity` is a plain number. Leave every price cell empty.

**Sheet "Conditions"**, when `conditions` is not empty: `topic`, `text_original`, `text_en`, `reference`, `page`.

**Sheet "About"**:

- the buyer's form: `form.name_original`, `form.name_en`, `form.number`, `form.page`;
- the currency rule: `currency.text_original`, `currency.text_en`, `currency.page`. If `currency` is null, write "Currency not stated in the schedule"; never assume one;
- the total rows the form asks for: each `label_original` and `label_en` of `total_rows`;
- `buyer`, `tender_reference` and `language` as given;
- if `partial` is true: "Lucius listed part of a long schedule. Price from the buyer's own form.";
- if `result.price_schedule.workbook` is present: the buyer's own price workbook asks for more prices than are listed here (`rows` against `listed`). Tell the team to price from the buyer's workbook;
- the analysis `app_url`, and when `result.notice` is present its `source_url` and `attribution`.

## 4. Finish

Give the file and say how many lines it has, the currency rule if stated, and the page of the buyer's form. Remind the user that prices go into the buyer's own form when the pack prescribes one.

## Errors

Errors come as JSON with `error` and `status`. On 402 the workspace has used its Lucius allowance: say the request did not run, repeat the `error` text and do not retry. For other errors, repeat the `error` text.

## Surface notes

- Claude: create the .xlsx with Claude's built-in Excel file creation. In Claude Code without it, write the file with a spreadsheet library such as openpyxl if it is installed.
