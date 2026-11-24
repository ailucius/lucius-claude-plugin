---
name: evidence-from-m365
description: Search the user's SharePoint and OneDrive for evidence that answers each requirement of a tender analysed by Lucius, and list the candidate files with links per requirement. Needs the Microsoft 365 connector. Use when the user asks to find evidence, supporting documents, certificates, CVs, case studies or past bids for a tender in SharePoint, OneDrive, Teams files or Microsoft 365.
---

# Evidence from SharePoint and OneDrive

Matches the requirements of a Lucius analysis to files in the user's own Microsoft 365. The search runs through the user's Microsoft 365 connector with the user's own access. Nothing found in Microsoft 365 is sent to Lucius.

## 0. Check the connector

This skill needs a Microsoft 365 connector that can search SharePoint and OneDrive files. If no such tool is available, stop and tell the user how to connect it: add the Microsoft 365 connector from the connector directory and sign in with their work account (an administrator of their organisation may need to allow it first). Then offer to continue.

## 1. Get the requirements

You need an analysis id: from earlier in the conversation, from a Lucius app link ending in `/dashboard/` and the analysis id, or by following the tender-check skill, steps 1 to 3.

Call `get_pack_analysis` with `id`, `deciding_only` true and `include_deliverables` false. Search for the deciding requirements first. If the user wants every requirement, read with `deciding_only` false and page with `result.requirements_next_offset`. If `result.requirements_locked` is true, only the deciding requirements are available.

## 2. Search, one requirement at a time

For each requirement:

1. Take two or three search terms from `requirement_plain` and from `original_text`: names of certificates and standards as written (for example an ISO number), the document the buyer asks for, the service or product. Use the original-language words too when the user's files may be in that language.
2. Search SharePoint and OneDrive with the connector. Search only for the requirement; do not browse unrelated folders.
3. Keep up to five candidates: file name, site or folder, last modified date and the link, all as the connector returned them.

## 3. Present the candidates

One block per requirement:

- `position` and `requirement_plain`, with `document` and `page` of the buyer's quote;
- the candidates, each with its link and the words that matched;
- a status: "candidate, not opened", "opened: answers it" (with the passage quoted), "opened: does not answer it", or "nothing found".

A candidate is only a candidate. Say a file answers a requirement only after you opened it and can quote the passage. Check dates on certificates and say when one looks expired, quoting the date.

## 4. Next steps

Offer to put the confirmed files in the Evidence column of the compliance matrix (compliance-matrix-excel) or into the response outline (response-draft-word).

If `result.requirements_locked` is true, end with `result.locked_reason` exactly as returned, and its link if it contains one, otherwise the analysis `app_url`.

## Rules

- Search the user's files only for the tender's requirements, and only with their request.
- Never pass Microsoft 365 content to a Lucius tool.
- Never invent a file, a link or a passage.

## Surface notes

- Claude: use Claude's Microsoft 365 connector (SharePoint and OneDrive search). Other assistants: use their Microsoft 365 or Microsoft Graph file search the same way.
