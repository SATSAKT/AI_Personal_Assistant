# Morning Brief

Purpose:
Runs a scheduled daily morning briefing: invokes the AI News Summarizer and Daily Share Market Snapshot sub-workflows via LangChain tool nodes, merges their outputs into one Telegram-friendly brief (5 bullets, ≤25 words each), and sends it to the user on Telegram.

Trigger:
Schedule Trigger (`scheduleTrigger`) — runs daily at **9:00** (hour set in `triggerAtHour: 9`).

Services Used:
- OpenAI (gpt-4.1-mini, via LangChain Chat Model)
- LangChain AI Agent (orchestrator with system prompt for brief format)
- Sub-workflows (LangChain Tool Workflow):
  - AI News Summarizer
  - Daily Share Market Snapshot
- n8n Data Table (`TelegramChatId`) — chat ID lookup for outbound messages
- Telegram Bot API (`MyAIPersonalAssistantBot`) — send text message

Flow:
Schedule Trigger → AI Agent (OpenAI + Call AI News Summarizer + Call Daily Share Market Snapshot) → TelegramChatId (Data Table get) → Send a text message (Telegram)

Agent behavior:
1. Call AI News Summarizer tool.
2. Call Daily Share Market Snapshot tool.
3. Combine both outputs into a single morning brief.
4. Return only the final formatted text (no tool names, no questions).

Output format (exactly 5 bullets):
- **AI News** — 2 bullets (≤25 words, relevant emojis)
- **Market Insights** — 2 bullets (≤25 words, finance emojis)
- **Daily Takeaway** — 1 bullet combining AI + market sentiment

Telegram message body: `={{ $('AI Agent').item.json.output }}`  
Chat ID: `={{ $json.chatId }}` from the `TelegramChatId` data table row.

Workflow inputs (sub-workflow tools):
Both Tool Workflow nodes use `mappingMode: defineBelow` with **empty** `value: {}` — no inputs are passed to child workflows. The sub-workflows run with their default `executeWorkflowTrigger` behavior only.

Benefits:
- Hands-free daily digest without opening Telegram or asking the bot
- Reuses existing AI News and Market Snapshot logic (no duplicate RSS/API code)
- Consistent, scannable format for Telegram (emoji sections, word limits)
- Centralized scheduling — change time in one Schedule Trigger node
