# AI News Summarizer

Purpose:
Aggregates the latest AI news from multiple RSS feeds, deduplicates and ranks them by date, then summarizes the top 5 stories into concise, Telegram-friendly bullet points using an OpenAI LLM.

Trigger:
Executed by Another Workflow (sub-workflow trigger — `executeWorkflowTrigger`)

Services Used:
- RSS Feed (TechCrunch AI, VentureBeat AI)
- OpenAI (gpt-4.1-mini, via LangChain Chat Model)
- LangChain AI Agent + Simple Memory (Buffer Window)

Flow:
RSS (TechCrunch + VentureBeat) → Merge → Deduplicate Articles → Sort (by date, desc) → Limit (5) → Prepare News Text → AI Agent (OpenAI + Memory)

Benefits:
- Saves reading time by surfacing the top 5 AI stories of the day
- Removes duplicate articles across feeds
- Produces Telegram-ready, emoji-friendly summaries (≤25 words per bullet)
- Reusable as a sub-workflow from any parent flow
