# Support Ticket Router – LangGraph Homework

Build a Google Colab notebook named `support_ticket_router.ipynb` that classifies TaskNest support tickets and routes them to three teams.

## Setup

Install dependencies with `!pip -q install langgraph langchain-openai`. Use `ChatOpenAI` with `gpt-4.1-mini` and `temperature=0`. Define the model name once. This classification task is simple enough for a low-cost model. The previous model did not support setting `temperature=0`, so use `gpt-4.1-mini` to keep that setting. Set `max_retries=0`.

Read the key from Colab Secrets using:

```python
import os
from google.colab import userdata

os.environ["OPENAI_API_KEY"] = userdata.get("OPENAI_API_KEY")
```

Avoid `getpass`, hardcoded credentials, and printing the key.

## State and classification

Define `TicketState` as a `TypedDict` with exactly four fields: `ticket`, `category`, `team`, and `reply`. Accept an initial state containing only `ticket`.

Use `with_structured_output` with a Pydantic model whose category is a `Literal` of `billing`, `technical`, and `general`. The `classify` node makes the only LLM call per ticket and returns only `{"category": ...}`.

Use these classification instructions:

```text
You are a customer support ticket classifier for TaskNest, a to-do app.

Read the customer's ticket and classify it into one of three categories:

- billing: Payments, invoices, refunds, and subscriptions.
- technical: Bugs, crashes, error messages, and login problems.
- general: Feature requests, general questions, thank-you messages, and anything else.

Choose the category based on the main problem described in the ticket, not just keywords.

If a ticket mentions more than one topic, choose the category that best matches the main issue.

Return only the category.
```

## Graph

Add exactly four nodes and one call to `add_conditional_edges`. Connect START to `classify`. The routing function `route(state)` only returns `state["category"]`. Each team node sets its fixed team and reply using plain Python and then connects to END.

| Category | Node | Team | Reply |
| --- | --- | --- | --- |
| billing | `billing_team` | Billing | Your ticket has been sent to our Billing team. |
| technical | `tech_team` | Technical Support | Your ticket has been sent to our Technical Support team. |
| general | `general_team` | General Support | Your ticket has been sent to our General Support team. |

Display the graph inside the notebook with `draw_mermaid_png()`.

## Examples

Include at least two original tickets per category and at least one ambiguous ticket covering multiple topics. Include tickets whose main issue differs from what a keyword alone would suggest. Use `graph.invoke` for each ticket and print its original text, category, team, and reply.

Keep the notebook simple and add short Markdown explanations before the main sections. Use no graph loops, tools, memory, or additional nodes. It should run top to bottom in a fresh Colab session once the secret is configured.

## Bonus – Interactive Multilingual Support Assistant

After the original graph is compiled, run the bonus cells in Google Colab. They enable Colab's widget manager and render a centered, responsive light interface using `ipywidgets` and `IPython.display`. The form starts with an empty Customer Ticket field and a hidden result area. **Route Ticket** submits the ticket, and **Clear** resets the form.

A deterministic Python pattern checks for Hebrew letters. Any match selects Hebrew, including in mixed-language tickets. All other tickets receive an English reply. Each nonblank submission invokes the existing compiled graph once and reads its team, category, and reply. The result card shows Detected Language, Assigned Team, and Customer Reply. Category stays internal.

English replies come from the existing team nodes. A fixed mapping supplies the Hebrew replies and displays them right to left. English replies display left to right. Neither detection nor translation calls the LLM. The interface uses the existing Colab Secrets configuration through the graph and needs no separate server or credentials.

The form shows validation for empty input, a processing message during routing, and a friendly error if routing fails. It makes no API calls when created. Rerunning the interface cell replaces the previous form and removes its click handlers.
