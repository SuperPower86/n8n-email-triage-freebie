# n8n Email Triage — free workflow

An n8n workflow that triages a Gmail inbox with an LLM: inquiries get an auto-reply and are saved as leads, support mails become tickets, billing goes to Slack, spam is archived — and anything the model is unsure about (confidence < 0.6) is labeled for a human instead of being acted on. Every classification is logged to Postgres so you can measure errors and improve the prompt.

This is a free sample from a pack of 10 n8n AI workflows I sell. It's the complete workflow, not a crippled demo — if it works for you as-is, keep it.

## Files
- `email-smart-classify.json` — import into n8n (Workflows → Import from File)
- Setup, testing, and troubleshooting: see the sections below in this README

## Requirements
n8n ≥ 1.52 · Gmail OAuth2 · OpenAI API key or any OpenAI-compatible local endpoint (Ollama works) · Postgres

## Notes
- Prompts inside the workflow are in Chinese; the logic is language-neutral, swap the prompt text and it behaves the same.
- First activation processes your existing inbox as new mail — test in manual mode first.

## The full pack
9 more in the same style (support bot, lead scoring, meeting minutes, CRM sync, report generator…): https://litoolkit.gumroad.com/l/opkdof

## What you need

- n8n ≥ 1.52 (self-hosted or cloud)
- A Gmail account connected via OAuth2 (with label write permission)
- An OpenAI API key, or any OpenAI-compatible local endpoint (Ollama works fine — classification does not need a big model; use temperature 0)
- Postgres for the log table (schema below). Zendesk and Slack branches are optional — delete them if you don't use them.

```sql
CREATE TABLE IF NOT EXISTS email_classifications (
  id BIGSERIAL PRIMARY KEY,
  message_id TEXT, sender TEXT, subject TEXT,
  category TEXT, confidence NUMERIC, summary TEXT, extract TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

## Setup

1. Import `email-smart-classify.json` in n8n (Workflows → ⋯ → Import from File) and replace the Gmail / Postgres / Slack / Zendesk credentials with your own.
2. Create a label in Gmail named `待人工分诊` (or rename it in the workflow to whatever you like, e.g. "needs-human-review"), then re-select it in the low-confidence node.
3. Point the Zendesk node at your own subdomain, or delete that branch.
4. Adjust the category definitions in the classifier prompt and the `0.6` threshold in the fallback node to match your team.
5. Test before activating. **Important:** the Gmail Trigger treats your whole current INBOX as new mail on first activation. Empty it first, or test in manual mode — otherwise it will happily auto-reply to old emails. Ask me how I know.

## Testing it

Send yourself 5 emails: a one-line inquiry, a fault report, an invoice request, a marketing mail, and one piece of gibberish. Run the workflow manually and check: 5 rows in the log table with sensible categories, the inquiry got a threaded reply, and the gibberish one got the human-review label.

## Honest notes

- The prompts inside the workflow JSON are in Chinese (that's how I run it). The logic is language-neutral — swap the prompt text for English and it behaves the same. The category names in your prompt just need to match the Switch node.
- If auto-replies show up as new emails instead of threaded replies, the `inReplyTo` field is empty — you're probably passing the thread ID instead of the message ID.
- Not on Gmail? Swap the trigger node for the IMAP trigger and the rest works as-is.
