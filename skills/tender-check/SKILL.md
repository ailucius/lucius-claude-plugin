---
name: tender-check
description: Check a public tender with Lucius. Find it or open it by link or id, analyse its documents, and give the bid call with the requirements that decide the bid, each quoted from the pack with its document and page. Use when the user shares a tender or notice link, a Lucius tender id or a Lucius app link, asks whether a tender is worth bidding on, asks what a tender requires, or asks to find tenders on a subject, in a country or closing soon.
---

# Tender check

This skill uses the Lucius tools `search_tenders`, `get_tender`, `analyze_pack` and `get_pack_analysis`. Use only what these tools return. Never add a requirement, a page, a date, a value or a contact that the tools did not give.

## 1. Find or open the tender

Work out what the user gave:

- **A Lucius tender id** (a uuid from an earlier search): call `get_tender` with `id`.
- **A Lucius app link** whose path ends in `/dashboard/` and an id: that id is an analysis id. Go to step 3 with it.
- **A public link** to a notice page or to a tender document (PDF, Word, Excel or ZIP): go to step 2 with `url`.
- **A subject** ("road maintenance in Ireland closing this month"): call `search_tenders` with `query`, and only when the user gave them, `countries`, `cpv_prefix` or `closing_within_days`. Results are open notices. Use `next_cursor` for the next page.
- **Files the user attached**: these tools take a tender id or a public link, not files. Say so, and tell the user to upload the files in the Lucius app and paste the analysis link from there.

Show each search result as one line: `title`, `buyer`, `country`, `deadline_at`, and `source`, with `source_url` as the link. Show `value` only when it is not null, with its `currency` and `value_label` exactly as given; never add a currency the result does not give. When `also_on` lists the same tender on another source, say so in a few words. Keep each notice's `attribution` with it. Then ask which tender to check.

For one tender from `get_tender`, show the same fields from `tender`, plus `status` and the `documents` the source lists (`title`, `url`, `type`). Keep `attribution`. If `analysis` is not null, this workspace has analysed the tender already: use `analysis.analysis_id` in step 3 and do not start a new analysis.

## 2. Start the analysis

Call `analyze_pack` with exactly one of `tender_id` or `url`.

- If the user asked to check or analyse this tender, start it. If they only searched, ask before you start: an analysis uses one of the workspace's monthly Lucius analyses. A tender the workspace already analysed, or the same link sent again within 24 hours, returns the existing analysis (`deduplicated` true) and uses none.
- Leave `verdict` at its default, which judges the bid call against the workspace's company profile. Only when the user reads the tender for another company, send `judge_as` with a short description they give you, or `verdict` "none" for no call.

The answer has `analysis_id`, `status` and `app_url`. When `documents_from` is present, the documents came from another notice of the same tender or from the buyer's portal: say which.

## 3. Wait for the result

Call `get_pack_analysis` with `id` set to the analysis id, `deciding_only` true and `include_deliverables` false.

- `status` "queued" or "processing": tell the user the `progress` and `message`, then check again after a short wait. These checks are free. An analysis can take several minutes. If it is still running after several checks, give the `app_url` and say the user can ask again later.
- `status` "failed": give the `reason` as returned and the `app_url`.
- `status` "complete": present the `result` as in step 4.

## 4. Present the result

Keep this order and keep it short.

1. **The tender**: `result.title`, `result.buyer`, `result.deadline`. When `result.notice` is present, give its `source_url` and keep its `attribution`.
2. **What was read**: if `result.document.is_tender` is false, give `result.document.note`, make no call, and stop after the coverage notes.
3. **The call**: `result.verdict.call`, `result.verdict.recommendation` and `result.verdict.confidence` exactly as returned, then `result.verdict.summary`. Add `judged_as_label` or `closed_on` when present.
   - `result.verdict.call` null: there is no call yet. Give `result.verdict.note`, which says how many deciding requirements are still open and what to do.
   - `result.verdict.stale` true: give the note; the company details changed after the call was made.
   - `result.verdict` null with `result.verdict_mode` present: the analysis was run without a call.
4. **The deciding requirements**: for each requirement with `deciding` true, give `requirement_plain`, then the quote from `original_text` in full and in its original language, then `document` and `page`. Say "about page N" when `page_is_approximate` is true. When `quote_verified` is false, add "quote not checked against the page". When the quote is not in English and `requirement_en` is present, give it as the English line. Then list `result.verdict.deciding_requirements` (each `question`, `kind`, `mandatory`, `answer`, with `clause`, `page_ref` and `source_doc`) when they add anything.
5. **Strengths and risks**: `result.verdict.strengths` and `result.verdict.risks` as returned.
6. **Coverage**: if `result.coverage.complete` is false, list `result.coverage.notes`. If `result.requirements_final` is false, say a second read is still running and more requirements may be added.
7. **Forms**: if `result.forms_status` is "ready", list each form's `name`, `name_en`, `document`, `page` and `mandatory`. Otherwise give `result.forms_note` when it is not null.
8. **Plan limits**: if `result.requirements_locked` is true, say these are the requirements that decide the bid and the full list is not included for this tender. Give `result.locked_reason` exactly as returned. If it contains a link, end with that link. If not, end with the analysis `app_url`, where the full analysis opens in the Lucius app.
9. **Link**: end with `app_url`.

When `result.requirements_next_offset` is not null, there are more deciding requirements: call `get_pack_analysis` again with `requirements_offset` set to it.

## Rules

- Give each quote from `original_text` exactly as returned, in full: never shorten it, drop a sentence or put an ellipsis inside it. Never replace a quote with a translation; give the English line beside it.
- Never invent a page, a document name, a deadline or a requirement. If a field is null, say it is not stated.
- Never state, estimate or suggest a bid price.
- Keep every `attribution` with the notice it belongs to.

## When a tool answers with an error

Errors come as JSON with `error` and `status`.

- 402: the workspace has used its Lucius allowance for now. Say the request did not run and repeat the `error` text, which says when it renews. Do not retry, and do not quote plans or prices.
- 403: this connection cannot analyse packs. Repeat the `error` text.
- 404: not found for this workspace. Check the id.
- 429: the daily result cap is reached. Repeat the `error` text.
- 400: repeat the `error` text. A link on a portal whose terms do not allow Lucius to fetch it is refused: the user can download the files and upload them in the Lucius app.
- 500 or 503: try once more after a short wait, then give the `error` text.

## Surface notes

- Claude: the Lucius tools come from the Lucius connector. If they are not available, ask the user to connect it from the plugin's Connectors tab (Claude Code: run `/mcp` and sign in to `lucius`).
- Claude Code: between status checks you may wait with a short sleep in the shell.
