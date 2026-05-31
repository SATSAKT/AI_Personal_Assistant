# Daily Share Market Snapshot

Purpose:
Fetches the latest Nifty50 index and top crypto (BTC, ETH) prices, formats them into a market snapshot message, and produces a short professional commentary via an OpenAI agent.

Trigger:
Executed by Another Workflow (sub-workflow trigger — `executeWorkflowTrigger`). Invoked by the MCP Server as the `Daily Share Market Snapshot` tool.

Services Used:
- Yahoo Finance (`query1.finance.yahoo.com/v8/finance/chart/^NSEI`) — Nifty50 price + previous close
- CoinGecko (`api.coingecko.com/v3/simple/price`) — BTC & ETH in USD/INR with 24h % change
- OpenAI (gpt-4.1-mini, via LangChain Chat Model)
- LangChain AI Agent + Simple Memory (Buffer Window, keyed on `$workflow.id`)

Flow:
Trigger → (Get Nifty50 ‖ Get Crypto in parallel) → Merge (combineByPosition) → Format Market Snapshot (JS code builds `market_snapshot` string with emojis) → AI Agent (OpenAI + Memory) → commentary output

Output:
- `market_snapshot` — pre-formatted text with NIFTY50 price + % change, BTC/ETH in USD/INR + 24h change
- AI Agent's final commentary — a short, professional market summary based on the snapshot

Benefits:
- One call returns both Indian equity (Nifty50) and crypto (BTC/ETH) data
- Free, key-less data sources (Yahoo Finance + CoinGecko)
- AI commentary adds qualitative context instead of just raw numbers
- Reusable as a sub-workflow / MCP tool from the Telegram assistant
