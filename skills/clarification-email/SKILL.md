---
name: clarification-email
description: Draft, never send, a clarification email to a tender buyer with numbered questions about requirements that are unclear or conflict with each other, each citing the buyer's words with document and page from a Lucius analysis. Use when the user asks to draft clarification questions, questions to the buyer or contracting authority, a request for clarification, or an Outlook email to the buyer about a tender.
---

# Clarification email to the buyer

Drafts an email with numbered questions for the buyer. Each question cites the buyer's own words with document and page. The skill only drafts: the user reviews and sends it themselves.

## 1. Get the requirements

You need an analysis id: from earlier in the conversation, from a Lucius app link ending in `/dashboard/` and the analysis id, or by following the tender-check skill, steps 1 to 3.

Call `get_pack_analysis` with `id` and `include_deliverables` false, and page with `result.requirements_next_offset` until all rows are read. If `result.requirements_locked` is true, only the deciding requirements are available: say the questions cover those only.

## 2. Find what to ask

Look through the requirements for:

- **unclear wording**: an undefined term, a scope that is not stated, a quantity or a standard that is missing;
- **conflicts**: two quotes on the same subject that say different things (a date, a quantity, a standard, a minimum figure), often in different documents;
- **missing documents**: a quote that names an annex, form or appendix that is not in `result.documents`.

Ask the user which of their own questions to add. Keep only questions the buyer can answer. Do not ask anything the pack already answers; check the other requirements first.

## 3. Find the channel and the deadline

Look in the requirements' `original_text` for how and by when the buyer takes questions (clarification, questions, enquiries, or the same words in the pack's language).

- Use a contact address, portal or deadline only when a quote states it. Cite its `document` and `page`.
- If no quote gives a contact, leave the To line empty and say so. Many buyers take questions only through their tender portal; point the user to the notice `source_url` when `result.notice` is present.
- Never guess an email address.

## 4. Draft the email

- **To**: the contact from step 3, or empty.
- **Subject**: "Request for clarification: " followed by `result.title`.
- **Opening**: one neutral sentence naming the tender as `result.title` and `result.buyer` give it.
- **Questions**, numbered. Each one: a short subject; the buyer's words quoted from `original_text` with `document` and `page`; the question in one or two plain sentences. For a conflict, quote both passages.
- **Close**: a neutral thank you, and the user's name and company left as placeholders for them to fill in.

Buyers often share clarification answers with all bidders. Write the questions so they do not reveal the user's own approach, weaknesses or pricing. Write in English unless the user asks for the pack's language; keep the quotes in their original language either way.

## 5. Finish

Show the draft as text the user can paste into Outlook or their portal, then list the sources (document and page per question) and the questioning deadline if a quote states one.

Never send the email. If a mail connector that can save drafts is connected and the user asks for it, save it as a draft only and tell them where it is.

## Errors

Errors come as JSON with `error` and `status`. On 402 the workspace has used its Lucius allowance: say the request did not run, repeat the `error` text and do not retry. For other errors, repeat the `error` text.

## Surface notes

- Claude: to save an Outlook draft, a connector must offer draft creation; the user's explicit request is required each time. Never call a send action.
