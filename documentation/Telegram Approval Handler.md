# Telegram Approval Handler

Purpose:
Handles the user's Telegram reply to an AI-generated LinkedIn draft (from the `AI Newsletter Auto Post` flow). Routes the reply to one of three actions: publish to LinkedIn, reject, or rewrite the draft with new instructions.

Trigger:
Telegram Trigger (`telegramTrigger`) on `MyAIPersonalAssistantBot` — fires on every message update.

Services Used:
- Telegram (`MyAIPersonalAssistantBot`) — trigger + status replies
- n8n Data Tables:
  - `GmailGeneratedPost` — read latest draft, upsert rewrites
  - `TelegramChatId` — recipient lookup
- LinkedIn (`AI Content Publisher` OAuth) — publishes the approved post as person `y8BBRZv01Y`
- OpenAI (gpt-4.1-mini, via LangChain AI Agent) — rewrites the draft for the `REWRITE` branch

Flow:
Telegram Trigger → Split messages (`message` = text before `:`, `original` = full text) → Switch on `message` value:

- **APPROVE** → Get Post (`GmailGeneratedPost`) → Limit (1) → Create a post (LinkedIn) → Get row(s)1 (`TelegramChatId`) → Send "Posted" to Telegram
- **REJECT** → Get ChatId (`TelegramChatId`) → Send "Post rejected. No action taken." to Telegram
- **REWRITE** → Rewrite Instructions (strips `REWRITE:` prefix from `original`) → Get Post1 (`GmailGeneratedPost`) → AI Agent (rewrites using `gpt-4.1-mini`) → Upsert row(s) into `GmailGeneratedPost` → Get row(s) (`TelegramChatId`) → Limit1 (1) → Send new draft with `APPROVE / REJECT / REWRITE` prompt

Switch Routing (`Switch` node):
- `message` equals `APPROVE` → publish branch
- `message` equals `REJECT` → reject branch
- `message` equals `REWRITE` → rewrite branch

Note: matching is `string equals` (case-sensitive). `Split messages` splits on `:`, so `REWRITE: do X` becomes `message = "REWRITE"`, while plain `REWRITE` (no colon) also matches. `APPROVE` / `REJECT` must be exact, single-word replies.

LLM Contract (Rewrite branch):
- System: "You are a LinkedIn content editor. Rewrite the LinkedIn post based on the user's instructions. Keep the content professional and engaging."
- User: `Original Post: <Post>` + `Rewrite Instructions: <text after REWRITE:>`
- Output: rewritten post text, used as the new caption and upserted back into `GmailGeneratedPost`.

Benefits:
- Closes the human-in-the-loop on the `AI Newsletter Auto Post` flow — same `APPROVE / REJECT / REWRITE` contract
- Free-form rewrite instructions via Telegram, no UI needed
- Rewritten drafts replace the stored post and re-enter the approval loop
- Reject path is fail-safe — no LinkedIn publish, no data mutation
