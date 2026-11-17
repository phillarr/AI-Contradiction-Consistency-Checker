# AI Contradiction / Consistency Checker

A scenario that reads customer emails about deliveries, checks the customer's claim against your order system (Google Sheets) and your CRM notes (Notion), and alerts your team on Slack when the claim conflicts with your records or needs a human look.

## The scenario

```
Gmail (new email) ──► OpenAI (extract the claim) ──► Google Sheets (find the order) ──► Notion (find the CRM note) ──► Router
                                                                                                                      ├─► Source missing: stop
                                                                                                                      └─► Source found ──► OpenAI (compare) ──► Router
                                                                                                                                                                 ├─► No action needed: stop
                                                                                                                                                                 └─► Needs review ──► Slack (alert)
```

## What it does

1. **Watch new emails** (Gmail): checks your inbox for unread emails.
2. **Extract the claim** (OpenAI, structured data): pulls out, only from what is written in the email:
   - `customer_name`
   - `order_id` (for example `ORD-2026-0847`)
   - `claim_text`: the customer's claim about delivery, in one short sentence
   - `claim_field`: `delivery_date` or `other`
3. **Find the order** (Google Sheets): looks up the order in an `Orders` sheet.
4. **Find the CRM note** (Notion): looks up the same order in a Notion database.
5. **Router**: if the order or the CRM note can't be found, the scenario stops.
6. **Compare** (OpenAI, structured data): compares the claim with the order record and the CRM note, and returns:
   - `contradiction_detected`: `true` or `false`
   - `conflicting_field`: for example `delivery_date`
   - `explanation`: one sentence, framed as "the claim conflicts with our records", never as "the customer is wrong"
   - `recommended_action`: `no_action`, `escalate` or `human_review`. When the order record and the CRM note disagree, or the note is vague, the result is ambiguous and goes to human review.
7. **Slack alert**: unless the action is `no_action`, posts a message with the claim, the order record, the CRM note, the conflict and the recommended action (🔴 contradiction detected or 🟠 human review needed).

> **The AI can make mistakes.** It can misread the email, pick the wrong order number, or see a contradiction where there is none (or miss one). A Slack alert is a prompt to check, not a verdict: always look at the records yourself before answering the customer.

## Requirements

- A Make.com account
- A Gmail account
- An OpenAI API key
- A Google Sheet with your orders
- A Notion database with your CRM notes
- A Slack workspace

## Data setup

**Google Sheet**: a sheet named `Orders`, with the order number in column A and the order details (including the delivery date) in the next columns.

**Notion database**: one page per order, with:

| Property | Type  | Content                                |
|----------|-------|----------------------------------------|
| Order    | Title | Order number, written exactly as in the sheet |
| Note     | Text  | Your CRM note about the order          |

## Installation

1. Download [`AI Contradiction - Consistency Checker.blueprint.json`](AI%20Contradiction%20-%20Consistency%20Checker.blueprint.json).
2. In Make, create a new scenario, open the **⋯** menu, choose **Import Blueprint** and select the file.
3. Reconnect each module to your own accounts:
   - **Gmail**: add your connection, and set the **From** filter to the sender(s) you want to watch (or clear it).
   - **OpenAI**: add a connection with your API key.
   - **Google Sheets**: pick **your** spreadsheet and sheet.
   - **Notion**: pick **your** database.
   - **Slack**: pick **your** channel.
4. Check the field mappings in the Slack message (order record columns and Notion note).
5. Click **Run once** and send yourself a test email, for example:
   > Hi, my order ORD-2026-0847 was supposed to arrive on March 3 but I still haven't received it. — Jane Doe
6. Turn on scheduling when you are happy with the result.

## Customizing

- **Claims**: the prompts focus on delivery dates. Adapt them to other claims (refund amounts, product versions, contract terms...).
- **Models**: the extraction step uses a small model, the comparison step a larger one. Change them in each OpenAI module.
- **Missing records**: the "Source is missing" route stops the scenario. Replace it with a Slack message so these emails are not silently ignored.
- **Alerts**: send the alert by email or create a ticket instead of posting on Slack.

## Privacy and security

- **Email contents, order records and CRM notes are sent to OpenAI** for processing. Only use this with data you are allowed to share with a third-party AI provider.
- **The Gmail module marks the emails it reads as read.** Turn off "Mark as read" if you don't want that.
- **API keys and OAuth tokens stay in your Make connections** and are never exported in the blueprint.
- **Emails are untrusted input.** Someone could write an email designed to trick the AI (prompt injection). The prompts tell the AI never to conclude that the customer is wrong, and a human reviews every alert.

## License

[MIT](LICENSE)
