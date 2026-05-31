# MCP Server

Purpose:
Central MCP (Model Context Protocol) server that exposes the assistant's sub-workflows as AI-callable tools. Any MCP client (e.g. the Telegram Main Interface) connects to this endpoint and invokes the right tool based on user intent.

Trigger:
MCP Server Trigger (`mcpTrigger`) — webhook at path `/mcp/afd51f77-a027-4fdf-a721-6642c900c4b5`.

Services Used:
- n8n MCP Server Trigger (LangChain)
- Tool Workflows (sub-workflows registered as `ai_tool`):
  - Daily Share Market Snapshot
  - AI News Summarizer
  - LinkedIn Post (inputs: `topic`, `post_type`, `tone` — auto-filled via `$fromAI(...)`)
  - Expense Logger (input: `text` — auto-filled via `$fromAI(...)`)
  - Budget Summary (input: `chatID` — auto-filled via `$fromAI(...)`)

Flow:
MCP Client (e.g. Telegram Main Interface) → MCP Server Trigger → AI Agent selects the matching tool workflow → tool workflow runs and returns its result to the client

Exposed Tools:
- **Daily Share Market Snapshot** — formatted market snapshot with optional AI commentary
- **AI News Summarizer** — summarised latest AI news (TechCrunch AI + VentureBeat AI)
- **LinkedIn Post** — generates / publishes LinkedIn posts from `topic`, `post_type`, `tone`
- **Expense Logger** — parses an expense message and logs amount/category/date
- **Budget Summary** — monthly budget overview keyed by `chatID`

Benefits:
- Single MCP endpoint exposing every assistant capability as a tool
- Decouples consumers (Telegram bot, other MCP clients) from each workflow's implementation
- Typed tool inputs via `$fromAI(...)` so the LLM fills arguments automatically
- New capabilities added by attaching another Tool Workflow node — no client changes required
