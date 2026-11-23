# Curious Minds club chatbot

Build a Google Colab notebook for Homework 4 using the class geography chatbot as the starting point.

## Changes from the class prompts

- Base chatbot (`promt1.docx`). Keep the Colab secret lookup, model, `add_messages` reducer, and one-node graph. Build the system message from the club in state.
- Checkpointer (`prompt2.docx`). Keep the per-thread config and pass only the new message plus the selected topic on each invoke. Use Supabase-backed `PostgresSaver` to store history and the active club across restarts.
- Gradio (`prompt3.docx`). Keep the title, short guidance, examples, and Colab launch. Use `gr.Blocks` for the club and saved conversation controls. Allocate a UUID per conversation and use Supabase to find threads after a reload or restart.

## Graph and storage

- Install `langgraph`, `langchain`, `langchain-openai`, `langgraph-checkpoint-postgres`, `psycopg[binary]`, and `gradio` in Colab.

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

- Title and short description. Explain the active club and show sample questions labeled by club.
- Club dropdown. Default to Astronomy. A user change keeps the current thread and persists the topic immediately for a conversation with messages. For an empty draft, use the new topic on its first message.
- Chat window. Display user and assistant messages from the latest saved state.
- Message box and Send button. Both Enter and Send submit only the new message and active topic using the current thread ID. Ignore blank messages and clear the box after a successful turn.
- New conversation button. Allocate a UUID, clear the chat and message box, deselect the saved conversation, and keep the selected club.
- Saved conversation dropdown. Build choices from Supabase checkpoints on page load and after changes. Use the saved club, first question and a short ID in each label. Selecting a conversation restores its thread ID, messages and club with `graph.get_state(config).values`. Do not show the separate test thread.
- Store the active thread ID in Gradio session state. Postgres is the source of truth for saved conversation history and topics. Serialize UI operations against the shared connection. Run in Colab and keep the Gradio share link private.

## Verification

Run these four turns on `test-1` before launching Gradio.

1. Cooking — “How do I make pancakes?” should get a recipe.
2. Cooking — “Who won the 2022 World Cup?” should be declined with a suggestion to switch to Football.
3. Football — “Who won the 2022 World Cup?” should get an answer.
4. Football — “And how long do I fry each pancake?” should be declined as a return to the previous topic.

Try a football-themed food request in Cooking and Football, and a follow-up about the old club after switching. Start a new thread and confirm its chat is empty. Restart the Colab runtime, rerun setup, then reopen the saved conversation and check its topic, history and next reply. Inspect the checkpoint tables in Supabase before writing the reflection.

Submit the final prompt and notebook in a public GitHub repository. Include a Colab link the lecturer can open and a four- to six-sentence reflection based on the live checks. Keep credentials and the Gradio share link out of the submission.

## Prompt revision — Gradio UI and UX

Redesign only the Gradio presentation for teenagers. Use a responsive workspace with the conversation and composer in the main panel. Group club selection, examples, saved conversations, and the New conversation button in a sidebar that stacks above chat on narrow screens.

Use a violet and teal palette, readable sans-serif typography, rounded panels and chat bubbles, generous spacing, and visible keyboard focus. Keep the five clubs in the labeled dropdown. Explain that switching clubs keeps the current conversation. Keep the saved conversation dropdown and short examples, give the empty chat an inviting prompt, and keep the primary Send button next to the message field.

Apply styling through the Gradio theme and CSS at launch. Preserve the existing component roles, callback wiring, Enter and Send behavior, thread handling and queue. Keep the LangGraph graph, model, system message, Supabase persistence, and lecturer's verification cells as specified above.
