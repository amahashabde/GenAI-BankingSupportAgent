# Multi-Agent Banking Customer Support Assistant

A multi-agent GenAI system for bank customer support. A **classifier agent** reads each customer message and routes it to the right **specialist agent**, which thanks the customer, opens a support ticket, or looks up a ticket's status. Every decision is logged to SQLite, and a built-in **evaluation** measures how accurately messages are classified.

Built with Python, LangChain, OpenAI (`gpt-4o-mini`), SQLite and Gradio.

## Why

Banks handle a high volume of support messages through fragmented systems that struggle to personalize responses or give timely status updates. This project shows how a small team of LLM-powered agents can sort incoming messages, take the right action automatically, and keep an auditable record of what happened.

## How it works

```
Customer message
      │
      ▼
Classifier agent (LLM, temperature 0)
      │
      ├── Positive Feedback ──► Positive feedback agent ──► personalized thank-you (LLM)
      │
      ├── Negative Feedback ──► Negative feedback agent ──► new 6-digit ticket in SQLite + apology
      │
      └── Query ─────────────► Query agent ──► extracts ticket # (regex) ──► status lookup in SQLite
      │
      ▼
Agent log (SQLite): message, classification, agent used, response, timestamp
```

| Agent | Role | Uses an LLM? |
|---|---|---|
| **Classifier agent** | Labels each message as Positive Feedback, Negative Feedback, or Query | Yes |
| **Positive feedback agent** | Writes a short, warm, personalized thank-you | Yes |
| **Negative feedback agent** | Generates a unique ticket number, saves the ticket, and apologizes | No (deterministic) |
| **Query agent** | Finds the ticket number in the message and returns its status | No (deterministic) |
| **Coordinator** | Runs the classifier, routes to the right agent, and logs the result | — |

**Design choice:** LLMs are used only where language understanding is needed (classification and tone). Ticket creation and lookup are plain Python and SQL, which keeps them fast, predictable, and easy to test.

## Features

| Tab | What it does |
|---|---|
| **1. API Key** | Enter your OpenAI key at runtime (never stored in code) |
| **2. Customer Support Agent** | Submit a message and see the response, classification, and the routing path through the agents |
| **3. Support Tickets** | View all tickets in the database |
| **4. Logs** | View every agent decision |
| **5. Evaluation** | Run a test set and get classification accuracy with pass/fail per case |

## Evaluation

The notebook includes a small labeled test set covering all three categories. It runs each message through the classifier, compares the result with the expected label, and reports accuracy along with a pass/fail table.

## Tech stack

- **Python**: SQLite (`sqlite3`), Pandas, regex
- **LangChain**: prompt templates and LCEL chains (`prompt | llm | parser`)
- **OpenAI**: `gpt-4o-mini`
- **Gradio**: multi-tab web interface
- **Google Colab**: runtime

## Getting started

1. Open `Banking-Support-Agent.ipynb` in **Google Colab** (or Jupyter).
2. Run the cell. Dependencies install automatically:
   ```
   gradio langchain langchain-openai pandas
   ```
3. Open the Gradio link that appears.
4. Paste your **OpenAI API key** in the *API Key* tab.
5. Try messages like:
   - "Thanks for resolving my credit card issue."
   - "My debit card replacement still has not arrived."
   - "Could you check the status of ticket 650932?"

The database starts with three sample tickets (650932, 123456, 987654) so status lookups work right away.

## Limitations and next steps

- **Larger evaluation set:** expand beyond the starter cases and track precision and recall per category
- **Structured output:** have the classifier return JSON with a confidence score, and send low-confidence messages to a human
- **Mixed messages:** handle messages that contain both a complaint and a question
- **Ticket lifecycle:** add status updates, assignment, and resolution notes
- **Security:** verify customer identity before revealing ticket details

## Author

**Ashish Mahashabde**: engineer and AI builder
