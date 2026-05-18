# Task 3 — AI Learning Bot (Telegram + n8n)

## What this is

An AI-powered Telegram learning assistant built with n8n. Users can send article URLs, receive AI-generated summaries, save materials, take quizzes, and get scored — all inside Telegram.

**Live bot:** [@vention_ai_challenge_2_bot](https://t.me/vention_ai_challenge_2_bot)

---

## Telegram Commands

| Command | Description |
|---|---|
| `/start` | Greets the user and explains how to use the bot |
| `/learn {url}` | Fetches the article at `{url}`, generates an AI summary with title, difficulty level, and key concepts, then saves the material |
| `/quiz` | Lists all saved learning materials so the user can pick one to be quizzed on |
| `/quiz_{material_index}` | Starts a quiz for the selected material — generates questions via AI and asks them one by one |
| `/answer_{material_index}_{answer_index}` | Submits the user's answer for the current question in the active quiz session |

---

## How I approached it

### 1. Generated an overall plan with ChatGPT

Before touching n8n, I asked ChatGPT to produce a full architectural plan for the bot. This covered:

- User flows (`/start`, `/learn`, `/quiz`)
- High-level module breakdown (Trigger → Router → Processor → AI Agent → Storage → Quiz Engine → Results)
- Expected data shapes and AI prompts

The plan is saved in [`plan.md`](./plan.md).

### 2. Implemented one block at a time

Rather than trying to build everything at once, I asked ChatGPT to give me instructions for **one module at a time**:

1. Telegram Trigger Layer
2. Command Router
3. Learning Material Processor (URL extract → fetch → clean → AI summary → save)
4. AI Teacher Agent
5. Persistence Layer (material storage)
6. Quiz Selection Flow
7. AI Examiner Agent
8. Quiz Engine (question-by-question interaction)
9. Results and Feedback

Each step produced concrete n8n node configurations, which I then followed to wire up the workflow.

### 3. Followed the ChatGPT output

I treated each block's instructions as a recipe and built it step by step in n8n.

---

## Edge cases ChatGPT missed (that I had to fix manually)

### Duplicate summary / quiz generation
ChatGPT did not account for idempotency. The bot was calling AI every time, even if the article had already been processed.

**Fix:** Added existence checks before triggering AI calls:
- Check if material already exists for the URL before calling the AI Teacher.
- Check if a quiz already exists for a material before calling the AI Examiner.

### Material–Quiz relationship
ChatGPT treated quiz answers as universal (not scoped to a specific material/quiz session). This caused wrong validation — answers for one quiz could bleed into another.

**Fix:** Made quiz answers explicitly tied to a specific `material_id` + `quiz_id`, not just the user.

---

## Why this task was very big for a challenge

This was the most complex task of the challenge. It required integrating four distinct systems from scratch:

| Area | Why it was a lot of work |
|---|---|
| **Telegram integration** | Back-and-forth message handling required 5 separate routes/webhooks to manage the full conversation state |
| **n8n workflow** | Each module is a separate subflow; wiring them together correctly took significant iteration |
| **Data storage** | Two related entities (material + quiz) with their own schemas, relationships, and CRUD operations |
| **AI integration** | Two AI agents (Teacher + Examiner) with structured JSON output contracts that had to be validated |

---

## Data Storage

Two n8n DataTable tables are used:

### `learning_materials`
| Field | Description |
|---|---|
| `userId` | Telegram chat ID of the user |
| `url` | URL of the submitted article |
| `title` | AI-extracted title |
| `difficulty` | `beginner` / `intermediate` / `advanced` |
| `summary` | AI-generated bullet-point summary |
| `content` | Cleaned article text (up to 5,000 characters) |

### `quiz_sessions`
| Field | Description |
|---|---|
| `userId` | Telegram user ID |
| `materialIndex` | ID of the related learning material |
| `question` | The quiz question text |
| `options` | Answer options (comma-separated) |
| `correctAnswer` | The correct answer text |

---

## Implementation Notes

### AI model
Both the AI Teacher and AI Examiner agents use **`gpt-4o-mini`**.

### Content truncation
After fetching and cleaning the article HTML, the content is **truncated to 5,000 characters** before being sent to the AI. This keeps token usage low and avoids context limit errors on large pages.

### Telegram event types
The workflow listens to two distinct Telegram event types:
- **`message`** — regular text commands (`/start`, `/learn`, `/quiz`)
- **`callback_query`** — fired when a user taps an inline keyboard button (`quiz_*`, `answer_*`)

Both are handled by the same Telegram Trigger node and routed via a Switch node.

---

## Workflow export

The full n8n workflow is exported in [`task_3.json`](./task_3.json). You can import it directly into any n8n instance.

---

## Tech stack

- **n8n** — workflow automation
- **Telegram Bot API** — user interface
- **OpenAI** — AI Teacher + AI Examiner agents
- **n8n built-in storage / HTTP nodes** — persistence and content fetching
