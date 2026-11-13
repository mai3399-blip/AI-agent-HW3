# Support Ticket Router – LangGraph Homework

Build a Google Colab notebook named `support_ticket_router.ipynb` that classifies TaskNest support tickets and routes them to three teams.

## Setup

Install dependencies with `!pip -q install langgraph langchain-openai`. Use `ChatOpenAI` with `gpt-6-luna` and `temperature=0`. Define the model name once. This classification task is simple enough for a low-cost model.

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
