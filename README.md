# Lucius for Claude

Lucius reads public tenders for bid teams. This plugin connects Claude to your Lucius workspace so you can find a tender, have Lucius read the whole tender pack, and get the bid call with the requirements that decide the bid, each quoted from the pack with its document and page. From there Claude turns the analysis into the files bid teams work in: an Excel compliance matrix, an Excel price schedule, a Word response outline, a list of evidence from your SharePoint and OneDrive, and a clarification email to the buyer.

## What is inside

| Skill | Command | What it does |
| --- | --- | --- |
| tender-check | /lucius:check | Finds or opens a tender, starts the Lucius analysis, and presents the call and the deciding requirements with quotes and pages. |
| bid-no-bid | /lucius:bid | A short bid or no-bid note: the call with its reasons from the deciding requirements. Never a price. |
| compliance-matrix-excel | /lucius:matrix | An .xlsx with one row per requirement: quote in the original language, English line, document, page, severity, and blank owner and status columns. |
| price-schedule-excel | /lucius:prices | An .xlsx of the buyer's price schedule lines, quantities, units and pages, with empty price columns. Says plainly when the pack has none. |
| response-draft-word | /lucius:draft | A .docx outline that answers each deciding requirement from your own documents and marks every gap. |
| evidence-from-m365 | /lucius:evidence | Searches your SharePoint and OneDrive for evidence per requirement and lists candidate files with links. Needs the Microsoft 365 connector. |
| clarification-email | /lucius:clarify | Drafts, never sends, numbered questions to the buyer about unclear or conflicting requirements. |

The commands run in Claude Code and Cowork. In Claude chat you can simply ask; Claude picks the skill that fits.

## Install

In Claude Code 2.1.275 or later:

```
/plugin install lucius --marketplace ailucius/lucius-claude-plugin
```

On an older version, add the marketplace first:

```
/plugin marketplace add ailucius/lucius-claude-plugin
/plugin install lucius@lucius-ai
```

## Sign in

The plugin uses the Lucius connector (https://api.ailucius.com/api/v1/mcp), the same server as the Lucius connector in the Claude directory. You need a Lucius workspace at https://ailucius.com.

- Claude chat and Cowork: open the plugin's Connectors tab, connect Lucius and sign in to your Lucius account.
- Claude Code: run /mcp, choose lucius and sign in.

Sign in as an owner or admin of the workspace, with your email address confirmed. Only an owner or admin can connect an app, because its requests use the workspace's allowance.

## Plans and allowance

The plugin uses your workspace's own Lucius allowance. Each analysis uses one of the workspace's monthly analyses, like an upload in the Lucius app; a tender the workspace has already analysed, or the same link sent again within 24 hours, uses none. Searches and reads count as requests; checking on an analysis while it runs is free. When the allowance is used, Claude tells you the request did not run and when the allowance renews.

On a free workspace you get the bid call and the requirements that decide the bid. The full requirements matrix, the price schedule and drafts come with a Pack for that tender or a plan, unlocked in the Lucius app. The plugin quotes no prices.

## Try it

- "Check this tender for us: <link to the notice or the tender documents>"
- "Find open tenders for school catering in Ireland closing in the next 30 days."
- "Should we bid on it? Give me a one-page bid or no-bid note."
- "Make the compliance matrix for this tender in Excel."
- "Put the price schedule in a spreadsheet."
- "Find our ISO certificates and reference projects in SharePoint for each deciding requirement."
- "Draft the clarification questions for the buyer."

## Data

- Claude sends Lucius what each tool needs: your search words and filters, a tender id, a public link to tender documents, and, when you ask for it, a short description of the company a call is judged for. Lucius fetches the tender documents from the public link or from the notice's own source.
- Analyses are stored in your Lucius workspace, where your team can open them in the app. Only your workspace can read them.
- Search results are public procurement notices: metadata, a short snippet and a link to the source notice. Each carries an attribution, the credit its source's licence asks for, and Claude keeps it with the notice.
- The Excel and Word files are created by Claude in your conversation. The plugin stores nothing itself.
- Microsoft 365 searches run through your own Microsoft 365 connector with your own access. Nothing found there is sent to Lucius.
- Emails are drafted only. The plugin never sends anything.

Privacy policy: https://ailucius.com/privacy. Terms: https://ailucius.com/terms.

## Limits

- Lucius serves public notices; the full notice text is not served.
- Links on portals whose terms do not allow Lucius to fetch their material are refused; download the files and upload them in the Lucius app instead. The tools take a tender id or a public link, not attached files.
- An analysis takes a few minutes. Claude checks on it and tells you when it is ready.

## Support

Questions and problems: https://ailucius.com/contact or contact@ailucius.com. Developer documentation: https://ailucius.com/developers.

## License

MIT. See LICENSE.
