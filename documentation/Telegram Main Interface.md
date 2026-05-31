# Telegram Main Interface

Purpose:
The main Telegram bot entry point. Receives user messages, routes them through an OpenAI-powered agent that picks the right sub-tool (AI News, Share Market, LinkedIn Post, Expense Logger, Budget Summary, or general web answer), and replies on Telegram.

Trigger:
Telegram Trigger (`telegramTrigger`) — listens for incoming `message` updates from the `MyAIPersonalAssistantBot` bot.

Services Used:
- Telegram (Trigger + Send message)
- OpenAI (gpt-4.1-mini, via LangChain Chat Model)
- LangChain AI Agent
- Simple Memory (Buffer Window, keyed per Telegram chat ID)
- MCP Client (custom MCP server endpoint for tool calls)

Flow:
Telegram Trigger → AI Agent (OpenAI Chat Model + Simple Memory + MCP Client tools) → Send a text message (Telegram)

Routed Capabilities (handled by the AI Agent's system prompt):
- AI News Summarizer — latest AI / OpenAI / Anthropic / GenAI updates
- Daily Share Market Snapshot — Nifty50, Sensex, crypto, mutual funds
- LinkedIn Post — generates LinkedIn-ready posts
- Expense Logger — logs and summarizes expenses
- Budget Summary — monthly budget / savings / overspending insights
- General Query — web/Google-backed answers for anything else
- Greeting / Help — friendly welcome message listing capabilities

Benefits:
- Single Telegram bot as a unified front-end for all personal-assistant workflows
- Per-chat conversational memory via Simple Memory keyed on `chat.id`
- Tools are pluggable via the MCP Client, so new capabilities can be added without changing the Telegram interface
- Intent routing handled by the LLM — no manual command parsing required
