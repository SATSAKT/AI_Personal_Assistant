# LinkedIn Post

Purpose:
Turns a free-text topic into a LinkedIn-ready post plus a matching banner image, sends both as a Telegram photo message with an approval prompt (`YES` / `REGEN` / `NO`), and stores the caption for the follow-up publish flow.

Trigger:
Executed by Another Workflow (sub-workflow trigger — `executeWorkflowTrigger`). Invoked by the MCP Server as the `LinkedIn Post` tool with input `{ topic, post_type, tone }`.

Services Used:
- OpenAI:
  - `gpt-4.1-mini` — Basic LLM Chain (topic parser, `json_object`), Generate LinkedIn Post, Generate Image Prompt
  - `gpt-image-2` — banner image generation (1024x1024, medium quality)
- Telegram — send photo with caption (`MyAIPersonalAssistantBot`)
- n8n Data Tables:
  - `TelegramChatId` — looked up to find the recipient `chatId`
  - `LinkedinPost` — caption is upserted after the photo is sent (used by the approval/publish follow-up flow)

Flow:
Trigger → Basic LLM Chain (parse topic into `{ topic, post_type, tone }`) → JSON output → parallel branches:
- Generate LinkedIn Post (OpenAI) → Get ChatId (Data Table) → Merge (port 0)
- Generate Image Prompt (OpenAI) → Generate an image (`gpt-image-2`) → Merge (port 1)

Then: Merge → Send a photo message (Telegram, caption = generated post + `YES/REGEN/NO` prompt) → Store Resume Data (caption) → Update row(s) in `LinkedinPost` data table.

LLM Contracts:
- **Topic parser** (Basic LLM Chain) — extracts strict JSON `{ topic, post_type, tone }`.
  - `post_type` ∈ `Achievement | Launch | Learning | Career Update | Project | Motivation | Hiring | Networking | AI | Tech | Other` (default `Other`)
  - `tone` ∈ `Professional | Inspirational | Casual | Technical | Excited` (default `Professional`)
- **Post generator** — strong hook → 3 insights → engagement question, professional tone, sparing emojis, no hashtag spam.
- **Image prompt** — Corporate / Minimalist / Dark blue / Modern / No text / Banner style.

Output:
A Telegram photo message to the user containing the AI-generated banner and the LinkedIn post text as the caption, followed by:
```
Reply:
YES = publish
REGEN = regenerate image
NO = cancel
```
The caption is stored in the `LinkedinPost` data table so a separate flow can resume on the user's `YES` / `REGEN` / `NO` reply.

Benefits:
- One Telegram message → full LinkedIn-ready post + matching banner image
- Inline approval loop (`YES/REGEN/NO`) keeps the user in control before anything is published
- Topic parser normalises freeform input, so post type & tone are usable downstream
- Reusable as a sub-workflow / MCP tool from the Telegram assistant
