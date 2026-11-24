---
name: response-draft-word
description: Draft a Word (.docx) response outline for a tender that answers each deciding requirement from a Lucius analysis, using only the user's own documents as evidence and marking every gap. Use when the user asks to draft a tender response, bid response, proposal outline, answer document or response to the requirements, or to start writing the bid in Word.
---

# Response outline in Word

Builds a .docx outline with one section per deciding requirement: the buyer's words with document and page, then a draft answer written only from the user's own material. Where the user's material does not answer a requirement, the section says so as a gap. This is a working draft for the bid team, not a submission.

## 1. Get the analysis

You need an analysis id: from earlier in the conversation, from a Lucius app link ending in `/dashboard/` and the analysis id, or by following the tender-check skill, steps 1 to 3.

## 2. Read the requirements

Call `get_pack_analysis` with `id`, `deciding_only` true and `include_deliverables` false. `status` must be "complete". While `result.requirements_next_offset` is not null, call again with `requirements_offset` set to it.

If the user wants every requirement covered, not only the deciding ones, read with `deciding_only` false instead. If `result.requirements_locked` is true, only the deciding requirements are available: say so and draft those.

## 3. Collect the user's evidence

Use only:

- files the user attached or shared in this conversation (company profile, past bids, certificates, CVs, method statements, policies);
- facts the user stated in this conversation;
- documents found with the evidence-from-m365 skill that the user confirmed or that you opened and read.

Ask for what is missing before drafting if there is little or nothing to draw on. Never use general knowledge about the user's company as evidence.

## 4. Build the document

**Title block**: "Response outline: " followed by `result.title`; `result.buyer`; `result.deadline`; "Working draft for the bid team. Not for submission."; the analysis `app_url`. When `result.notice` is present, add its `source_url` and keep its `attribution`. If `result.requirements_locked` is true, add: "Covers the requirements that decide the bid only."

**One section per requirement**, in `position` order:

1. Heading: `requirement_plain`.
2. The buyer's words: `original_text` as a quote in its original language, then "Source: " with `document` and `page` ("about page N" when `page_is_approximate` is true). Add `requirement_en` as the English line when the quote is not in English.
3. "Our response": a draft answer built only from the evidence in step 3. After each statement, name the source in brackets: the file name, and the page or section when known.
4. "Evidence to attach": the user's documents that support the answer.
5. Where the evidence does not answer the requirement, write "GAP:" followed by what the requirement asks for, in its own words, and what kind of evidence would answer it. Never fill a gap with an invented certificate, number, name, date, reference project or price.

**Forms to return**: when `result.forms_status` is "ready", list each form's `name`, `name_en`, `document`, `page` and `mandatory`.

**Open questions**: requirements that are unclear or conflict with each other. Suggest the clarification-email skill for them.

**Coverage**: if `result.coverage.complete` is false, list `result.coverage.notes`, so the team knows which files were not read in full.

Use real Word headings so the outline shows in the navigation pane. Keep the buyer's quotes in a distinct quote style.

## 5. Finish

Give the file and a short count: sections drafted from evidence, sections with gaps.

If `result.requirements_locked` is true, end with `result.locked_reason` exactly as returned. If it contains a link, end with that link. If not, end with the analysis `app_url`, where the full requirements list can be opened in the Lucius app. Never write a different link and never quote a price.

## Rules

- Never state that the bid complies with a requirement. The team decides that.
- Never invent evidence. A gap marked as a gap is the right answer.
- Keep the buyer's quotes exactly as returned.

## Errors

Errors come as JSON with `error` and `status`. On 402 the workspace has used its Lucius allowance: say the request did not run, repeat the `error` text and do not retry. For other errors, repeat the `error` text.

## Surface notes

- Claude: create the .docx with Claude's built-in Word file creation. In Claude Code without it, write the file with a library such as python-docx if it is installed.
