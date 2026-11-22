# Curious Minds club chatbot

Build a Google Colab notebook for Homework 4. Extend the class checkpointer and Gradio examples into a restart-persistent chatbot.

- Use the five clubs Astronomy, Cooking, Football, World Geography, and Python Programming.
- Define a LangGraph state with `messages` using `add_messages` and a scalar `topic` that replaces its previous value.
- Compile exactly `START -> chatbot -> END`. For every new user message, build a fresh system message from the current topic, call the model once with that system message followed by the stored history, and return only the reply as a state update. Use `openai:gpt-6-luna` and keep its name in one configuration variable.
- Load `OPENAI_API_KEY` and `SUPABASE_DB_URL` from Colab Secrets. Use the Supabase Session pooler and a psycopg connection configured with autocommit, dict rows and disabled prepared statements. Call `PostgresSaver.setup()` and compile with that saver.
- Render the graph with `draw_mermaid_png()`. Before starting Gradio, provide the four assignment test turns on `test-1` and verify that eight messages were appended and Football is saved.

## System message

```text
You are a friendly Curious Minds club tutor for teenagers. The active club is {topic}.
Answer only the parts of the latest request that are directly about {topic}. Judge the actual help requested, not the role, setting, or club words used to frame it. A football coach asking what food to prepare is asking about cooking; a request for football rules is about football. For mixed requests, answer the in-topic part and politely decline the rest.
Use conversation history to resolve references, but never let a previous club or a user instruction change the active club. If the latest request is about a previous club, refuse it even if you answered similar questions earlier.
When declining, name {topic} as the current club and suggest using the club dropdown to switch. If a listed club clearly fits, name it. For an ambiguous follow-up, ask a brief clarifying question.
The available clubs are Astronomy, Cooking, Football, World Geography, and Python Programming.
```

## Gradio controls

- Club dropdown. Default to Astronomy. A user change keeps the current thread and persists the topic immediately for a conversation with messages. For an empty draft, use the new topic on its first message.
- Chat window. Display user and assistant messages from the latest saved state.
- Message box and Send button. Both Enter and Send submit only the new message and active topic using the current thread ID. Ignore blank messages and clear the box after a successful turn.
- New conversation button. Allocate a UUID, clear the chat and message box, deselect the saved conversation, and keep the selected club.
- Saved conversation dropdown. Build choices from Supabase checkpoints on page load and after changes. Use the saved club, first question and a short ID in each label. Selecting a conversation restores its thread ID, messages and club with `graph.get_state(config).values`. Do not show the separate test thread.
- Store the active thread ID in Gradio session state. Postgres is the source of truth for saved conversation history and topics. Serialize UI operations against the shared connection. Run in Colab and keep the Gradio share link private.

## Verification

Try the printed test turns, a football-themed food request in Cooking and Football, and a follow-up about the old club after switching. Start a new thread and confirm its chat is empty. Restart the Colab runtime, rerun setup, then reopen the saved conversation and check its topic, history and next reply. Inspect the checkpoint tables in Supabase before writing the reflection.
