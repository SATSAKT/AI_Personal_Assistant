# AI Personal Assistant

A collection of [n8n](https://n8n.io/) workflows that act as a personal AI assistant — chat with it on Telegram, log expenses, get market and AI-news briefs, and let it draft + auto-publish LinkedIn posts from AI newsletters with human-in-the-loop approval.

## Architecture

The project is split into three flows:

### Flow 1 — On-demand assistant (Telegram + MCP)

A Telegram bot routes user requests through an **MCP server** that exposes the individual tool workflows. The agent decides which tool to call based on the user message.

```
Telegram message
   └─> Telegram Main Interface (agent + memory)
         └─> MCP Server
               ├─> AI News Summarizer       (RSS → LLM bullets)
               ├─> Daily Share Market Snapshot (Nifty50 + crypto + AI commentary)
               ├─> Expense Logger           (LLM parse → Google Sheets)
               ├─> Budget Summary           (Sheets → category totals → LLM advice)
               └─> LinkedIn Post            (LLM post + DALL·E image → Telegram approval)
```

### Flow 2 — Autonomous newsletter → LinkedIn

A Gmail trigger watches the inbox for AI newsletters and turns them into LinkedIn drafts. The user approves, rejects, or asks for a rewrite from Telegram, and the approved post is published.

```
Gmail (therundown.ai / techcrunch.com)
   └─> AI Newsletter Auto Post (LLM draft → store in GmailGeneratedPost → Telegram prompt)
         └─> user replies APPROVE / REJECT / REWRITE: <instructions>
               └─> Telegram Approval Handler
                     ├─ APPROVE  → LinkedIn Create a post → Telegram "Posted"
                     ├─ REJECT   → Telegram "Post rejected"
                     └─ REWRITE  → LLM rewrite → re-prompt for approval
```

### Flow 3 — Scheduled morning brief

A separate workflow runs on a daily schedule (9:00) and pushes a combined AI + market digest to Telegram without user input.

```
Schedule Trigger (9:00)
   └─> Morning Brief (agent + AI News + Market Snapshot tools)
         └─> TelegramChatId lookup → Send message
```

## Repository layout

```
documentation/   # Markdown description of each workflow
workflows/
  Flow1/         # On-demand Telegram/MCP assistant
  Flow2/         # Newsletter → LinkedIn auto-post + approval handler
  Flow3/         # Scheduled morning brief
```

## Workflows

| Flow | Workflow | Documentation | Source |
|------|----------|---------------|--------|
| 1 | Telegram Main Interface | [doc](documentation/Telegram%20Main%20Interface.md) | [json](workflows/Flow1/TelegramInterface.json) |
| 1 | MCP Server | [doc](documentation/MCP%20Server.md) | [json](workflows/Flow1/MCP%20Server.json) |
| 1 | AI News Summarizer | [doc](documentation/AI%20News%20Summarizer.md) | [json](workflows/Flow1/AI%20News%20Summarizer.json) |
| 1 | Daily Share Market Snapshot | [doc](documentation/Daily%20Share%20Market%20Snapshot.md) | [json](workflows/Flow1/Daily%20Share%20Market%20Snapshot.json) |
| 1 | Expense Logger | [doc](documentation/Expense%20Logger.md) | [json](workflows/Flow1/Expense%20Logger.json) |
| 1 | Budget Summary | [doc](documentation/Budget%20Summary.md) | [json](workflows/Flow1/Budget%20Summary.json) |
| 1 | LinkedIn Post | [doc](documentation/LinkedIn%20Post.md) | [json](workflows/Flow1/LinkedIn%20Post.json) |
| 2 | AI Newsletter Auto Post | [doc](documentation/AI%20Newsletter%20Auto%20Post.md) | [json](workflows/Flow2/AI%20Newsletter%20Auto%20Post.json) |
| 2 | Telegram Approval Handler | [doc](documentation/Telegram%20Approval%20Handler.md) | [json](workflows/Flow2/Telegram%20Approval%20Handler.json) |
| 3 | Morning Brief | [doc](documentation/Morning%20Brief.md) | [json](workflows/Flow3/Morning%20Brief.json) |

## Services & credentials

You'll need accounts/credentials configured in n8n for:

- **OpenAI** — `gpt-4.1-mini` (chat + post generation), `gpt-image-2` (LinkedIn banners)
- **Telegram Bot API** — bot token for the assistant (`MyAIPersonalAssistantBot` in the exports)
- **Gmail OAuth2** — inbox polling for Flow 2
- **Google Sheets OAuth2** — `Personal Finance Tracker` workbook for expense logging
- **LinkedIn OAuth2** — author the auto-published post (`AI Content Publisher`)
- **RSS feeds** — TechCrunch AI, VentureBeat AI (no auth)

The exported JSONs reference credentials by n8n credential ID/name only — no secrets are committed.

## n8n Data Tables

The flows persist intermediate state in three n8n Data Tables:

- `TelegramChatId` — chat ID lookup for sending outbound messages
- `GmailGeneratedPost` — latest LinkedIn draft produced from a newsletter
- `LinkedinPost` — captions for on-demand LinkedIn posts (from the Flow 1 `LinkedIn Post` workflow)

## Importing into n8n

1. In n8n, **Workflows → Import from File** and pick a JSON from `workflows/Flow1/`, `workflows/Flow2/`, or `workflows/Flow3/`.
2. Open each red-flagged node and reconnect it to your own credentials.
3. Recreate the Data Tables listed above (matching column names) in the same project.
4. For Flow 1, start with `TelegramInterface` and `MCP Server` — the rest are called as sub-workflows.
5. For Flow 2, both workflows must be active for the approval loop to work.
6. For Flow 3, activate `Morning Brief` and ensure Flow 1 sub-workflows (AI News Summarizer, Daily Share Market Snapshot) are imported.
