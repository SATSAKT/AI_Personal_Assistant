# AI Newsletter Auto Post

Purpose:
Watches Gmail for AI newsletter emails (e.g. The Rundown AI, TechCrunch), turns each one into a LinkedIn-ready draft via an OpenAI agent, stores the draft, and sends it to Telegram with an approval prompt (`APPROVE` / `REJECT` / `REWRITE: ...`).

Trigger:
Gmail Trigger (`gmailTrigger`) — polls the `INBOX` label every minute.

Services Used:
- Gmail (`sathisheee1297` OAuth) — inbox polling
- OpenAI (gpt-4.1-mini, via LangChain Chat Model + AI Agent)
- n8n Data Tables:
  - `GmailGeneratedPost` — upserts the generated post text
  - `TelegramChatId` — looked up to find the recipient `chatId`
- Telegram — sends the draft + approval prompt (`MyAIPersonalAssistantBot`)

Flow:
Gmail Trigger → If (sender filter) → Edit Fields (`subject`, `body`, `sender`) → AI Agent (OpenAI gpt-4.1-mini) → Upsert row(s) in `GmailGeneratedPost` → Get row(s) from `TelegramChatId` → Send a text message (Telegram)

Sender Filter (`If` node):
Forwards only emails whose `From` **equals** `therundown.ai` **OR** `techcrunch.com` (string equals, case sensitive). Note: Gmail's `From` field is typically of the form `"Name <user@therundown.ai>"`, so this strict-equals check may need to be widened to `contains` if real emails are being missed.

LLM Contract (AI Agent):
- Reads `subject` + `body` of the newsletter
- Extracts the most interesting AI news/insight
- Writes a LinkedIn-ready post: hook → short paragraphs → closing question → relevant hashtags
- Output: post text only, no extra wrapping

Telegram Message Format:
```
🚀 AI Newsletter LinkedIn Draft

━━━━━━━━━━

Subject:
<email subject>

━━━━━━━━━━

Output:
<LinkedIn draft>

━━━━━━━━━━

Reply with:

APPROVE
REJECT
REWRITE: your instructions
```

Benefits:
- Zero-touch capture of AI newsletter content into LinkedIn drafts
- Human-in-the-loop approval via Telegram before anything is posted
- Draft persistence in `GmailGeneratedPost` enables a separate publish/follow-up flow keyed by the post text
- Reusable pattern for new newsletter sources — just extend the sender filter
