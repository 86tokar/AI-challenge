# Telegram Learning Bot — Task 3 (n8n)

A Telegram bot that turns any web article into a structured study summary and a 5-question quiz. Built entirely in n8n with two AI roles (Teacher and Examiner), no extra services or paid subscriptions.

## What it does

- **`/start`** — shows help and the list of commands.
- **`/learn <url>`** — fetches the article, the **Teacher AI** extracts a structured summary (title, difficulty, main concepts, 5–7 key points, short overview) and saves the material to per-user storage.
- **`/quiz`** — lists the user's saved topics as inline buttons. After a topic is picked, the **Examiner AI** generates 5 multiple-choice questions specific to the chosen material and sends them one by one with `A / B / C / D` buttons. After the last answer the bot returns a score and per-question feedback (with explanations for wrong answers).

## How to use the bot

1. Open the bot in Telegram (link in submission form).
2. Send `/start` to see the help.
3. Send `/learn https://en.wikipedia.org/wiki/React_(JavaScript_library)` (or any other article URL).
4. Wait a few seconds — the bot replies with a structured summary and an inline button **"📝 Take quiz on this material"**.
5. Either tap that button right away, or later send `/quiz` to choose from the list of all materials you've added.
6. Answer the 5 questions by tapping `A / B / C / D` under each one.
7. After the last answer the bot sends:
   - your score (`X/5`, percent),
   - a per-question review with correct answers and explanations for the ones you got wrong.


