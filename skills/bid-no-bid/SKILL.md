---
name: bid-no-bid
description: Give the bid or no-bid call on a tender from a Lucius analysis, with the reasons taken from the requirements that decide the bid, each quoted with document and page. Never gives a price. Use when the user asks whether to bid, go or no-go, bid or no bid, should we tender for this, or for a bid decision note for their manager.
---

# Bid or no bid

A short decision note built from the bid call Lucius made and the requirements that decide the bid. Works on every Lucius plan: the call and the deciding requirements are always included.

## 1. Get the analysis

Follow the tender-check skill, steps 1 to 3, to find the tender and get a complete analysis. Read it with `deciding_only` true and `include_deliverables` false.

## 2. Check that there is a call

- `result.document.is_tender` false: there is no call. Give `result.document.note`.
- `result.verdict` null and `result.verdict_mode` present: the analysis was run without a call. Say so; offer to run a new analysis with the default `verdict` (this uses one of the workspace's monthly analyses).
- `result.verdict.call` null: Lucius makes no call yet. Give `result.verdict.note`, which says how many deciding requirements are open (`open_requirements`) and what to do. Then list the deciding requirements as the questions to answer.
- `result.verdict.stale` true: give `result.verdict.note`; the company details changed since the call was made.
- `closed_on` present: the tender has closed. Say so first.

## 3. Write the decision note

1. **Call**: `result.verdict.call`, `result.verdict.recommendation` and `result.verdict.confidence` exactly as returned. Add `judged_as_label` when present, so the reader knows whose fit was judged.
2. **Why**: the deciding requirements, each with `requirement_plain`, the quote from `original_text` in full with `document` and `page`, and, from `result.verdict.deciding_requirements`, the `question`, whether it is `mandatory`, and the `answer` as returned ("yes", "no", or not answered yet). Keep the order Lucius returns, which puts the pass or fail gates first.
3. **What would change the call**: the `mandatory` gates answered "no", then the gates not answered yet. Answers are given in the Lucius app.
4. **Risks and strengths**: `result.verdict.risks` and `result.verdict.strengths` as returned. Add `result.verdict.competition` when it is not null.
5. **Deadline**: `result.deadline`.
6. **Next steps**: the compliance matrix, a response outline, or clarification questions, as the user needs.
7. **Link**: the analysis `app_url`.

## Rules

- Never give, estimate or suggest a price, a rate, a margin or a contract value as a reason for the call.
- Never make a call of your own or change the one Lucius returned. If the user disagrees, show which deciding requirement their view depends on.
- Give each quote from `original_text` exactly as returned, in full: never shorten it, drop a sentence or put an ellipsis inside it. Never invent a page.

## Errors

Errors come as JSON with `error` and `status`. On 402 the workspace has used its Lucius allowance: say the request did not run, repeat the `error` text and do not retry. For other errors, repeat the `error` text.
