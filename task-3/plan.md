# AI Learning Bot — n8n Workflow Plan

## 1. Project Overview

This project is an AI-powered Telegram learning assistant built with n8n.

The bot should allow users to:

- Send educational article URLs
- Receive AI-generated summaries
- Save learning materials
- Generate quizzes from saved materials
- Answer quizzes interactively in Telegram
- Receive score and explanations

The goal is not just to build a chatbot, but a small stateful AI learning system.

---

## 2. Main User Experience

### 2.1 Start Command

User sends:

```text
/start
```

Bot replies with a short explanation:

```text
Welcome!
Send me a learning article using:

/learn https://example.com/article

Later, use /quiz to test yourself.
```

---

### 2.2 Learn Command

User sends:

```text
/learn https://example.com/article
```

Bot should:

1. Extract the URL
2. Fetch article/page content
3. Clean the content
4. Send the content to AI Teacher
5. Generate summary and difficulty level
6. Save the material
7. Return summary to the user

Example bot response:

```text
Title: React Hooks Guide

Difficulty: Intermediate

Summary:
• Hooks allow function components to use state
• useState stores local state
• useEffect handles side effects

You can now start a quiz with /quiz
```

---

### 2.3 Quiz Command

User sends:

```text
/quiz
```

Bot should:

1. Load saved learning materials for this user
2. Show a list of topics
3. Let user choose one topic
4. Generate quiz questions using AI Examiner
5. Ask questions one by one
6. Validate user answers
7. Show final score and explanations

---

## 3. High-Level Architecture

The workflow can be divided into these modules:

1. Telegram Trigger Layer
2. Command Router
3. Learning Material Processor
4. AI Teacher Agent
5. Persistence Layer
6. Quiz Selection
7. AI Examiner Agent
8. Quiz Engine
9. Results and Feedback

---

## 4. Module A — Telegram Trigger Layer

### Purpose

This is the entry point of the system.

### n8n Node

Use:

```text
Telegram Trigger
```

### It should listen for:

- `/start`
- `/learn`
- `/quiz`
- callback queries from inline buttons

Telegram sends different types of events:

- regular text messages
- callback queries when users click buttons

The workflow must handle both.

---

## 5. Module B — Command Router

### Purpose

The router decides what should happen depending on the user input.

### Possible n8n nodes

Use one of:

- Switch node
- IF node
- Code node if routing becomes more complex

### Routing logic

| Input Type | Destination |
|---|---|
| `/start` | Welcome Flow |
| `/learn URL` | Learning Material Flow |
| `/quiz` | Quiz Selection Flow |
| Inline button callback | Quiz/Answer Handling Flow |

### Why this is important

Without a clear router, the workflow becomes messy very quickly.

The router is the traffic controller of the whole system.

---

## 6. Module C — Learning Material Processing

This module handles the `/learn` command.

---

### 6.1 Extract URL

User sends:

```text
/learn https://example.com/article
```

The workflow must separate:

- command: `/learn`
- URL: `https://example.com/article`

If URL is missing, bot should reply:

```text
Please send a URL like this:

/learn https://example.com/article
```

---

### 6.2 Fetch Article Content

### n8n Node

Use:

```text
HTTP Request
```

The node should request the URL and retrieve page content.

### Problem

Raw HTML pages contain a lot of noise:

- menus
- ads
- headers
- footers
- scripts
- CSS
- unrelated links

So the content should be cleaned before sending it to AI.

---

### 6.3 Clean Content

The system should extract useful text from the page.

Possible cleaning steps:

- Remove HTML tags
- Remove scripts and styles
- Remove navigation/footer content if possible
- Trim very long content
- Keep only meaningful article text

### Important

The AI summary must be based on the actual page content.

If we send garbage HTML to AI, the summary and quiz will be poor.

---

### 6.4 Send Content to AI Teacher

The AI Teacher is responsible for explaining the material.

### AI Teacher responsibilities

The Teacher should:

- Identify the title/topic
- Summarize the content
- Detect difficulty level
- Extract key concepts
- Format the result clearly

### Expected structured output

The AI should return JSON like this:

```json
{
  "title": "React Hooks Guide",
  "difficulty": "intermediate",
  "summary": [
    "Hooks allow function components to use state.",
    "useState stores local component state.",
    "useEffect handles side effects."
  ],
  "concepts": [
    "useState",
    "useEffect",
    "component lifecycle"
  ]
}
```

### Why JSON matters

Structured output is important because later workflow steps need to save and reuse the data.

Plain text is harder to parse.

---

### 6.5 Save Learning Material

The material must be saved so the user can use it later.

This is required because the task needs persistence between sessions.

### Recommended storage for MVP

Use:

```text
n8n Data Store
```

Alternative options:

- SQLite
- Postgres
- Airtable
- Google Sheets
- Notion database

For this exercise, n8n Data Store is probably the simplest.

### Save this data

Each saved material should contain:

```json
{
  "materialId": "unique-id",
  "userId": "telegram-user-id",
  "url": "https://example.com/article",
  "title": "React Hooks Guide",
  "difficulty": "intermediate",
  "summary": [],
  "concepts": [],
  "rawContent": "...",
  "createdAt": "timestamp"
}
```

### Important fields

- `userId` is needed to show only that user's materials
- `materialId` is needed to select a topic for quiz
- `rawContent` or cleaned content is needed to generate quiz questions later

---

### 6.6 Send Summary Back to Telegram

After saving, bot sends the summary to the user.

Example:

```text
Saved successfully!

Title: React Hooks Guide
Difficulty: Intermediate

Key points:
• Hooks allow function components to use state
• useState stores local state
• useEffect handles side effects

Use /quiz when you want to test yourself.
```

Optional button:

```text
[ Start Quiz ]
```

---

## 7. Module D — Quiz Selection

This module handles `/quiz`.

---

### 7.1 Load Saved Materials

When user sends:

```text
/quiz
```

Workflow should:

1. Get Telegram user ID
2. Search saved materials by this user ID
3. Return available topics

If no materials exist, bot replies:

```text
You do not have saved materials yet.

Send one first:

/learn https://example.com/article
```

---

### 7.2 Show Topic Buttons

Bot sends inline buttons:

```text
Choose a topic:
```

Example buttons:

```text
[ React Hooks ]
[ Docker Basics ]
[ Kubernetes Intro ]
```

Each button should contain callback data.

Example callback data:

```text
quiz_select:material_123
```

This tells the workflow which material the user selected.

---

## 8. Module E — AI Examiner Agent

The AI Examiner generates quiz questions.

### Input

The Examiner receives:

- title
- summary
- concepts
- cleaned article content
- difficulty level

### Responsibilities

The Examiner should:

- Generate quiz questions
- Provide multiple choice answers
- Mark the correct answer
- Provide explanations

### Expected output

```json
{
  "questions": [
    {
      "question": "What does useState return?",
      "options": {
        "A": "Only the current state",
        "B": "Current state and setter function",
        "C": "Only a setter function",
        "D": "A lifecycle method"
      },
      "correctAnswer": "B",
      "explanation": "useState returns the current state value and a function to update it."
    }
  ]
}
```

### Recommended number of questions

For MVP:

```text
5 questions
```

That is enough to demonstrate the feature without making the workflow too complex.

---

## 9. Module F — Quiz Engine

This is the interactive part of the system.

The quiz engine manages:

- current question
- selected answer
- score
- progress
- final result

---

### 9.1 Why Quiz State Is Needed

Telegram does not remember where the user is in the quiz.

Every message or button click is a separate event.

So the workflow must store quiz progress manually.

---

### 9.2 Quiz Session Data

Create a temporary quiz session object.

Example:

```json
{
  "quizId": "quiz_001",
  "userId": "telegram-user-id",
  "materialId": "material_123",
  "questions": [],
  "currentQuestionIndex": 0,
  "score": 0,
  "answers": [],
  "status": "in_progress",
  "createdAt": "timestamp"
}
```

This can also be stored in n8n Data Store.

---

### 9.3 Send First Question

Bot sends:

```text
Question 1 of 5

What does useState return?
```

Inline buttons:

```text
A. Only the current state
B. Current state and setter function
C. Only a setter function
D. A lifecycle method
```

Each button has callback data.

Example:

```text
quiz_answer:quiz_001:0:A
quiz_answer:quiz_001:0:B
quiz_answer:quiz_001:0:C
quiz_answer:quiz_001:0:D
```

---

### 9.4 Handle User Answer

When user clicks answer button:

1. Telegram sends callback query
2. Workflow detects it
3. Workflow loads quiz session
4. Compares selected answer with correct answer
5. Updates score
6. Saves user answer
7. Moves to next question

---

### 9.5 Continue Until Final Question

If there are more questions:

- increment current question index
- send next question

If it was the last question:

- finish quiz
- calculate score
- show results

---

## 10. Module G — Results and Feedback

After the final question, bot sends final result.

Example:

```text
Quiz complete!

Score: 4/5
Percentage: 80%
```

Then show explanations.

Example:

```text
Question 1: Correct
Your answer: B
Correct answer: B

Explanation:
useState returns the current state and a setter function.
```

For wrong answers:

```text
Question 3: Incorrect
Your answer: A
Correct answer: C

Explanation:
useEffect is used for handling side effects in React components.
```

---

## 11. Storage Design

For MVP, use two logical stores.

---

### 11.1 Learning Materials Store

Stores permanent materials.

Fields:

```json
{
  "materialId": "string",
  "userId": "string",
  "url": "string",
  "title": "string",
  "difficulty": "string",
  "summary": "array",
  "concepts": "array",
  "content": "string",
  "createdAt": "string"
}
```

---

### 11.2 Quiz Sessions Store

Stores temporary quiz sessions.

Fields:

```json
{
  "quizId": "string",
  "userId": "string",
  "materialId": "string",
  "questions": "array",
  "currentQuestionIndex": "number",
  "score": "number",
  "answers": "array",
  "status": "string",
  "createdAt": "string",
  "finishedAt": "string"
}
```

---

## 12. AI Roles

The task should clearly separate AI roles.

---

### 12.1 Teacher AI

The Teacher explains and summarizes.

Input:

```text
Cleaned article content
```

Output:

```json
{
  "title": "...",
  "difficulty": "...",
  "summary": [],
  "concepts": []
}
```

---

### 12.2 Examiner AI

The Examiner tests the user.

Input:

```text
Saved material content + summary
```

Output:

```json
{
  "questions": []
}
```

---

## 13. Suggested n8n Workflow Structure

You can build it as one big workflow at first.

Later it can be split into smaller workflows.

---

### Option A — One Workflow MVP

Single workflow handles:

- Telegram Trigger
- command routing
- learn flow
- quiz flow
- answer validation

This is simpler for the exercise.

---

### Option B — Multiple Workflows

Separate workflows:

1. Telegram Router Workflow
2. Learn Material Workflow
3. Quiz Generator Workflow
4. Quiz Answer Handler Workflow

This is cleaner but more complex.

Recommended for this exercise:

```text
Start with Option A.
```

---

## 14. Step-by-Step Build Order

Do not build everything at once.

---

### Phase 1 — Telegram Bot Basics

Goal:

- Connect Telegram bot to n8n
- Handle `/start`

Result:

```text
User sends /start
Bot replies with welcome message
```

---

### Phase 2 — Learn Flow Without Database

Goal:

- Handle `/learn URL`
- Fetch URL content
- Send content to AI
- Return summary

No persistence yet.

This proves the basic AI flow works.

---

### Phase 3 — Add Persistence

Goal:

- Save summarized material
- Store user ID
- Store material title, URL, summary, content

Result:

```text
User can save learning materials
```

---

### Phase 4 — Quiz Generation

Goal:

- Load one saved material
- Send it to AI Examiner
- Generate 5 quiz questions

No interactive quiz yet.

Just confirm that valid JSON questions are generated.

---

### Phase 5 — Topic Selection

Goal:

- `/quiz` shows saved materials
- User selects one with inline button

Result:

```text
Bot knows which material user wants to quiz on
```

---

### Phase 6 — Interactive Quiz

Goal:

- Send first question
- Handle answer button
- Send next question
- Track score

This is the hardest phase.

---

### Phase 7 — Final Result

Goal:

- Calculate score
- Show correct/incorrect answers
- Show explanations

---

## 15. Important Technical Challenges

---

### 15.1 Article Extraction

Web pages are messy.

Bad extraction leads to bad summaries and bad quizzes.

For MVP, simple extraction is okay.

For better quality, use an article extractor later.

---

### 15.2 State Management

The quiz requires memory.

The workflow must store:

- current question
- selected answers
- score
- session status

This is probably the biggest concept in the task.

---

### 15.3 Telegram Callback Queries

Inline buttons produce callback query events.

These are different from normal text messages.

The router must handle both.

---

### 15.4 AI JSON Reliability

AI may sometimes return invalid JSON.

To reduce this:

- Ask AI explicitly for JSON only
- Use fixed schema
- Avoid extra explanation
- Validate output before using it

---

## 16. Error Handling Ideas

### Missing URL

```text
Please send a URL after /learn.
Example:
/learn https://example.com/article
```

### Invalid URL

```text
This does not look like a valid URL. Please try again.
```

### Failed Article Fetch

```text
I could not read this page. Please try another URL.
```

### No Saved Materials

```text
You do not have saved materials yet.
Send one using /learn URL.
```

### Broken AI Response

```text
I could not generate a valid result. Please try again.
```

---

## 17. MVP Scope

For the first version, keep it simple.

### Include

- Telegram bot
- `/start`
- `/learn`
- AI summary
- Save material
- `/quiz`
- Topic selection
- 5-question quiz
- Score and explanations

### Exclude for now

- User accounts outside Telegram
- Admin panel
- Advanced analytics
- Spaced repetition
- Multiple quiz types
- Payment system
- Complex database design

---

## 18. Final Mental Model

Think about the system like this:

```text
Telegram is the interface.
n8n is the workflow brain.
AI Teacher explains.
AI Examiner tests.
Data Store remembers.
Quiz Engine manages progress.
```

---

## 19. Main Success Criteria

The exercise should demonstrate that the system can:

- Receive a URL
- Understand article content
- Summarize it using AI
- Save the material
- Generate quiz questions from saved content
- Interact with the user in Telegram
- Track quiz state
- Return score and explanations

---

## 20. Recommended First Implementation

Start with the smallest useful flow:

```text
Telegram /start
Telegram /learn URL
HTTP Request
Clean content
AI Teacher summary
Telegram response
```

After this works, add storage and quiz features step by step.
