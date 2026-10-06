# Email RAG Agent

An AI-powered **Email Retrieval-Augmented Generation (RAG) Agent** that connects to Gmail, indexes email content into a vector database, and uses an agentic workflow to retrieve and analyze relevant emails before generating grounded responses.

The system is designed to make large email collections searchable through natural-language queries while reducing hallucinations by grounding responses in retrieved email content.

---

## Overview

Searching through hundreds or thousands of emails manually can be time-consuming.

This project combines:

* **Gmail** for email ingestion
* **Qdrant** for vector storage and semantic retrieval
* **Sentence Transformers** for embeddings
* **LangGraph** for agentic workflow orchestration
* **Groq LLMs** for response generation
* **RAG** for grounding answers in retrieved email data

Instead of sending every email directly to an LLM, the system first retrieves the most relevant emails and then provides that context to the model.

### Example

A user can ask:

> "What emails did I receive about internship opportunities?"

The system:

1. Understands the user's query.
2. Searches the indexed email collection.
3. Retrieves the most relevant emails.
4. Passes the retrieved context to the LLM.
5. Generates an answer based on the retrieved evidence.

---

# Key Features

* Gmail email ingestion
* Semantic email search
* Retrieval-Augmented Generation
* Agentic workflow using LangGraph
* Vector search using Qdrant
* Local Qdrant persistence
* LLM-powered email analysis
* Context-aware responses
* Environment-based secret management
* Docker support
* Modular Python backend
* Support for continuously ingesting new emails

---

# Architecture

```text
                    ┌──────────────────┐
                    │      Gmail       │
                    │   Email Inbox    │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │  Email Ingestion │
                    │     Pipeline     │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Text Processing  │
                    │ & Metadata       │
                    │ Extraction       │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │    Embedding     │
                    │      Model       │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      Qdrant      │
                    │  Vector Database │
                    └────────┬─────────┘
                             │
                        User Query
                             │
                             ▼
                    ┌──────────────────┐
                    │    LangGraph     │
                    │  Agent Workflow  │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Semantic Search  │
                    │ & Retrieval      │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      Groq        │
                    │       LLM        │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Grounded Answer  │
                    └──────────────────┘
```

---

# How the System Works

The application consists of two major pipelines:

```text
1. Email Ingestion Pipeline
2. Query & RAG Pipeline
```

---

## 1. Email Ingestion Pipeline

The ingestion pipeline is responsible for reading emails from Gmail and converting them into searchable vector representations.

### Step 1 — Connect to Gmail

The application connects to the Gmail mailbox using the configured Gmail credentials.

The system retrieves email information such as:

* Sender
* Recipient
* Subject
* Date
* Email body
* Message metadata

---

### Step 2 — Process Email Content

Raw email content is cleaned and converted into text suitable for retrieval.

Metadata is preserved along with the email content so that retrieved results can be associated with the original email.

---

### Step 3 — Generate Embeddings

The processed email text is converted into numerical vector representations using a sentence-transformer embedding model.

Conceptually:

```text
Email Text
    ↓
Embedding Model
    ↓
Vector Representation
```

These vectors allow the system to perform semantic similarity searches.

---

### Step 4 — Store Vectors in Qdrant

The generated embeddings and associated email metadata are stored in Qdrant.

Each stored record contains information that allows the application to retrieve the original email context when a relevant query is received.

```text
Email
  │
  ├── Embedding
  ├── Subject
  ├── Sender
  ├── Date
  └── Content
        │
        ▼
      Qdrant
```

---

### Step 5 — Continuous Email Ingestion

The ingestion component can be used to backfill existing mailbox messages and monitor the mailbox for newly arriving emails.

This allows the vector database to remain synchronized with the mailbox.

---

# 2. Query & RAG Pipeline

When the user asks a question, the agent follows a retrieval and generation workflow.

### Step 1 — User Query

The user provides a natural-language question.

Example:

```text
Find emails related to AI internships.
```

---

### Step 2 — Agent Workflow

The query enters the LangGraph workflow.

LangGraph is used to define and orchestrate the different stages of the agent.

```text
User Query
    ↓
LangGraph Agent
    ↓
Retrieval
    ↓
Context Processing
    ↓
LLM
    ↓
Response
```

---

### Step 3 — Semantic Retrieval

The query is converted into an embedding and searched against the vectors stored in Qdrant.

The system retrieves the most relevant email records.

For example:

```text
Query:
"AI internship opportunities"

             ↓

        Vector Search

             ↓

┌──────────────────────────────┐
│ Email: AI Engineer Intern    │
│ Email: ML Internship         │
│ Email: AI Automation Role    │
└──────────────────────────────┘
```

---

### Step 4 — Context Construction

The retrieved emails are converted into context for the language model.

Instead of allowing the LLM to answer solely from its internal knowledge, the system provides relevant email information as context.

```text
User Query
     +
Retrieved Email Context
     ↓
     LLM
```

---

### Step 5 — LLM Response Generation

The retrieved context is passed to the configured Groq-hosted LLM.

The model generates a natural-language response based on the retrieved email information.

This makes the system more suitable for questions where the answer depends on the user's own email data.

---

# Agent Workflow

The application uses **LangGraph** to structure the agent workflow.

A simplified workflow looks like:

```text
                ┌───────────────┐
                │   User Query  │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Query Analysis│
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │   Retrieval   │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Context Build │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │   Groq LLM    │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Final Answer  │
                └───────────────┘
```

This architecture allows additional agent nodes and decision-making logic to be introduced as the project evolves.

---

# Tech Stack

| Component            | Technology                     |
| -------------------- | ------------------------------ |
| Programming Language | Python                         |
| Agent Framework      | LangGraph                      |
| LLM                  | Groq                           |
| Vector Database      | Qdrant                         |
| Embeddings           | Sentence Transformers          |
| Email Provider       | Gmail                          |
| RAG                  | Retrieval-Augmented Generation |
| Containerization     | Docker                         |
| Configuration        | Environment Variables          |

---

# Project Structure

```text
email-rag-agent/
│
├── agent.py
├── main.py
├── common.py
├── ingest.py
├── check_discord.py
│
├── docker-compose.yml
├── requirements.txt
├── .env.example
├── .gitignore
├── README.md
│
└── ...
```

### Main Files

#### `agent.py`

Contains the main AI agent and RAG workflow.

Responsibilities include:

* Query processing
* Retrieval
* Agent orchestration
* Context construction
* LLM interaction
* Response generation

---

#### `ingest.py`

Handles email ingestion and indexing.

Responsibilities include:

* Gmail connection
* Email retrieval
* Email processing
* Embedding generation
* Qdrant insertion
* New email ingestion

---

#### `common.py`

Contains shared utilities and reusable functionality used across the application.

---

#### `main.py`

Application entry point responsible for starting the application workflow.

---

#### `check_discord.py`

Utility script used for checking/handling the project's external integration or notification workflow.

---

#### `docker-compose.yml`

Provides Docker-based configuration for running the required services.

---

# Setup

## 1. Clone the Repository

```bash
git clone https://github.com/selvasri58/Email-RAG-Agent.git
cd Email-RAG-Agent
```

---

## 2. Create a Virtual Environment

### Windows

```powershell
python -m venv venv
venv\Scripts\activate
```

### Linux / macOS

```bash
python3 -m venv venv
source venv/bin/activate
```

---

## 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

# Environment Variables

Create a `.env` file in the project root.

Use `.env.example` as the template.

Example:

```env
GROQ_API_KEY=your_groq_api_key

GROQ_MODEL=your_groq_model

GMAIL_ADDRESS=your_gmail_address

GMAIL_APP_PASSWORD=your_gmail_app_password

QDRANT_URL=your_qdrant_url

QDRANT_API_KEY=your_qdrant_api_key
```

**Never commit `.env` to GitHub.**

The `.gitignore` file is configured to exclude it.

---

# Gmail Configuration

For Gmail access, use a dedicated **Google App Password** rather than your normal Gmail password.

The mailbox account must have the appropriate authentication configuration enabled.

Store the generated app password only inside your local `.env` file.

---

# Qdrant Configuration

The project supports Qdrant for vector storage.

Qdrant can be used through:

* Local storage
* Qdrant server
* Qdrant Cloud

For local development, the project can use the configured local Qdrant storage or Docker-based setup.

---

# Running the Project

## Step 1 — Start Required Services

If using Docker:

```bash
docker compose up -d
```

---

## Step 2 — Ingest Emails

Run the ingestion pipeline:

```bash
python ingest.py
```

This retrieves emails, processes them, generates embeddings, and stores the resulting vectors in Qdrant.

---

## Step 3 — Start the Application

Run:

```bash
python main.py
```

The application can then be used to query the indexed email collection.

---

# Example Queries

The agent can be used for questions such as:

```text
Find emails related to AI internships.

What internship opportunities have I received recently?

Show me emails from recruiters.

Find emails mentioning Python and AI.

What companies contacted me about technical roles?

Summarize the latest emails related to job opportunities.

Find emails containing information about interview schedules.
```

The exact supported queries depend on the configured agent workflow and retrieval implementation.

---

# RAG Pipeline

The core RAG architecture can be summarized as:

```text
                EMAIL DATA
                    │
                    ▼
             Text Processing
                    │
                    ▼
              Embeddings
                    │
                    ▼
                 Qdrant
                    │
                    │
              User Question
                    │
                    ▼
               Embedding
                    │
                    ▼
            Similarity Search
                    │
                    ▼
           Relevant Email Data
                    │
                    ▼
             Context Building
                    │
                    ▼
                Groq LLM
                    │
                    ▼
             Final Response
```

---

# Why RAG?

A traditional LLM does not automatically have access to a user's private mailbox.

RAG provides a mechanism for connecting an LLM with external, application-specific information.

In this project:

```text
LLM Knowledge
      +
Private Email Data
      ↓
Context-Aware Response
```

The retrieval layer provides the model with relevant information from the user's own email collection.

---

# Security Considerations

This project works with potentially sensitive email information.

Important security practices:

* Never commit `.env`.
* Never expose Gmail app passwords.
* Never expose API keys.
* Do not upload private email datasets to public repositories.
* Use environment variables for credentials.
* Restrict access to production vector databases.
* Rotate credentials if they are accidentally exposed.

The repository intentionally excludes:

```text
.env
qdrant_storage/
__pycache__/
*.pyc
```

---

# Future Improvements

Potential improvements include:

* Better hybrid retrieval
* Email threading and conversation reconstruction
* Metadata-aware filtering
* Date-based email filtering
* Sender-based filtering
* Improved reranking
* Streaming responses
* Authentication and user management
* Multi-user mailbox support
* Gmail push notifications
* Advanced agent tool calling
* Conversation memory
* Evaluation framework for RAG quality
* Observability and tracing
* Production deployment

---

# Learning Outcomes

This project demonstrates practical experience with:

* Retrieval-Augmented Generation
* Vector databases
* Semantic search
* LLM application development
* Agentic AI workflows
* LangGraph
* Gmail API/IMAP-based data ingestion
* Embedding models
* Qdrant
* Prompt engineering
* Context construction
* Python backend development
* Docker-based deployment

---

# Author

**TusharKamble**

B.Tech — Artificial Intelligence & Machine Learning

GitHub: `https://github.com/tusharkamble2127`

---

# License

This project is intended for educational and portfolio purposes.
