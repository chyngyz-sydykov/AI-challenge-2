# Task 3 — AI Learning Bot

An AI-powered Telegram learning assistant built with n8n. Send article URLs, get AI summaries, and test yourself with auto-generated quizzes.

**Live bot:** [@vention_ai_challenge_2_bot](https://t.me/vention_ai_challenge_2_bot)

---

## How to Use the Bot

### Step 1 — Start the bot

Open Telegram and send:

```
/start
```

The bot will greet you and explain available commands.

---

### Step 2 — Learn from an article

Send any article URL using the `/learn` command:

```
/learn https://example.com/some-article
```

The bot will:
1. Fetch the article content
2. Clean and extract the text
3. Generate an AI summary with title, difficulty level, and key concepts
4. Save the material to your personal list
5. Reply with the summary

---

### Step 3 — Start a quiz

Once you have at least one saved material, send:

```
/quiz
```

The bot will show your saved topics as buttons. Tap a topic to begin.

---

### Step 4 — Answer questions

The bot asks 5 multiple-choice questions one at a time. Tap a button to select your answer for each question.

---

### Step 5 — See your results

After the last question, the bot shows your score and explains the correct answer for each question.

---

## Running the Workflow Yourself

To run this workflow on your own n8n instance:

1. Import `task_3.json` into n8n (**Workflows → Import from file**).
2. Create a Telegram bot via [@BotFather](https://t.me/BotFather) and copy the token.
3. In n8n, add a **Telegram credential** with your bot token.
4. Add an **OpenAI credential** with your API key.
5. Set the Telegram Trigger node to use your new credential.
6. Activate the workflow.
7. Open your bot in Telegram and send `/start`.

---

## Project Files

| File | Description |
|---|---|
| `task_3.json` | n8n workflow export — import this into n8n |
| `report.md` | Full development report (approach, decisions, issues) |
| `plan.md` | Architectural plan generated before implementation |
