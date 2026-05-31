# Expense Logger

Purpose:
Parses a natural-language expense message (e.g. "spent 250 on lunch at office") with an OpenAI LLM, appends a structured row to a Google Sheets expense tracker, and returns a monthly totals summary grouped by category.

Trigger:
Executed by Another Workflow (sub-workflow trigger — `executeWorkflowTrigger`). Invoked by the MCP Server as the `Expense Logger` tool with input `{ "text": "..." }`.

Services Used:
- OpenAI (gpt-4.1-mini, via LangChain `chainLlm` with `json_object` text format)
- Google Sheets — `Personal Finance Tracker` spreadsheet, `Expenses` sheet (read + append)

Flow:
Trigger → Basic LLM Chain (extracts `amount`, `category`, `note`, `date` as JSON) → JSON output (parse) → Edit Fields (typed assignment) → Append row in sheet → Get row(s) in sheet → Calculate current month total → response

LLM Extraction Contract:
- Allowed categories: `Food`, `Travel`, `Shopping`, `Entertainment`, `Bills`, `Health`, `Other` (unknown → `Other`)
- `date` in `YYYY-MM-DD`; defaults to today if not mentioned
- Strict JSON: `{ amount: number, category: string, note: string, date: "YYYY-MM-DD" }`

Output:
A formatted text response, for example:
```
✅ Expense logged successfully
💰 Total spent: ₹12,500
🍔 Food: ₹4,000
🚕 Travel: ₹2,500
...
```
Per-category totals are computed only over rows in the current month/year, with category emojis (`Food 🍔`, `Travel 🚕`, `Shopping 🛍️`, `Entertainment 🎬`, `Bills 💡`, `Health 🏥`, `Other 📌`).

Benefits:
- Logs expenses from a single chat message — no manual data entry
- Persistent record in Google Sheets, available for further analysis
- Returns immediate monthly summary so the user sees running totals after every log
- Reusable as a sub-workflow / MCP tool from the Telegram assistant
