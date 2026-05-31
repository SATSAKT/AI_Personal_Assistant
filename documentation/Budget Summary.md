# Budget Summary

Purpose:
Reads the user's expense history from Google Sheets, aggregates current-month spending per category, and asks an OpenAI model to return a short personal-finance summary (3 insight lines + 1 saving tip).

Trigger:
Executed by Another Workflow (sub-workflow trigger — `executeWorkflowTrigger`). Invoked by the MCP Server as the `Budget Summary` tool with input `{ "chatID": "..." }`.

Services Used:
- Google Sheets — `Personal Finance Tracker` spreadsheet, `Expenses` sheet (read)
- OpenAI (gpt-4.1-mini, via LangChain `openAi` "Message a model" node)

Flow:
Trigger → Get row(s) in sheet → Group by category (JS code, current month/year only) → Message a model (OpenAI, finance-advisor system prompt) → response

Aggregation Detail:
- Categories aggregated: `Food`, `Travel`, `Shopping`, `Entertainment`, `Bills`, `Health`, `Other`
- Rows are filtered to those whose `date` falls in the *current month and year*
- Emits `{ totals: { <category>: amount, ... }, totalSpent }`

LLM Contract:
- System: "You are a personal finance advisor."
- User: the JSON-stringified per-category totals
- Response: 3 short insight lines + 1 actionable saving tip, friendly tone

Benefits:
- One call returns a categorised view of the current month's spending
- AI commentary turns raw numbers into actionable advice
- Reusable as a sub-workflow / MCP tool from the Telegram assistant

Note:
The `chatID` workflow input is declared on the trigger but is not currently referenced inside the workflow body — present for forward compatibility with chat-scoped responses.
