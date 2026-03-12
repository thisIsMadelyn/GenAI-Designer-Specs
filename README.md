# System Designer Assistant
### AI-powered Java Spring Boot Architecture Tool

---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [What Was Built](#2-what-was-built)
3. [Architecture & Design Decisions](#3-architecture--design-decisions)
4. [Project Structure](#4-project-structure)
5. [Agent Pipeline](#5-agent-pipeline)
6. [API Reference](#6-api-reference)
7. [Setup & Run Guide](#7-setup--run-guide)
8. [Environment Variables](#8-environment-variables)
9. [Mock Mode vs Real Mode](#9-mock-mode-vs-real-mode)
10. [Docker Deployment](#10-docker-deployment)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Project Overview

**System Designer Assistant** is an AI-powered chatbot that takes a project description from a developer and automatically generates a complete Java Spring Boot system design including:

- Requirements analysis (functional, non-functional, constraints, scale)
- Architecture recommendation (Microservices vs Monolith with tradeoff explanation)
- MySQL database schema with ERD diagram (Mermaid)
- REST API design with Spring Security JWT configuration
- DevOps Enginner (Docker Compose configuration)
- Testing Manager (Unit and Integration Testing) 

The system uses **FastAPI** as the backend, **LangGraph** for multi-agent orchestration, **OpenAI GPT-4o** as the AI model, and **MySQL** for conversation history storage.

---

## 2. What Was Built

### Backend (Python / FastAPI)

| File | Purpose |
|------|---------|
| `main.py` | FastAPI app entry point, CORS, lifespan (DB init) |
| `models/schemas.py` | All Pydantic models (requests, responses, agent outputs) |
| `db/database.py` | SQLAlchemy async engine, ORM table, DB connection |
| `db/queries.py` | save_message, load_history, get_all_sessions |
| `agents/orchestrator.py` | LangGraph graph — routes and coordinates all agents |
| `agents/architect.py` | Agent 1: extracts requirements from user input |
| `agents/backend_layer.py` | Agent 2: designs system architecture |
| `agents/database.py` | Agent 3: designs MySQL schema and ERD |
| `agents/devops.py` | Agent 4: designs REST endpoints, JWT config, Docker |
| `agents/llm_factory.py` | Hardcoded responses for testing without API key |
| `agents/system_analyst.py` | Hardcoded responses for testing without API key |
| `agents/testing.py` | Hardcoded responses for testing without API key |
| `api/chat_router.py` | FastAPI routes: `/api/chat` and `/api/chat/design` |

### Frontend (React) — In Progress

| File | Purpose |
|------|---------|
| `src/hooks/useChat.js` | Session management, API calls |
| `src/components/MessageBubble.js` | Chat message rendering with markdown |
| `src/components/ChatInput.js` | Text input with Enter-to-send |
| `src/pages/MainPage.js` | Main layout — form panel + results panel |

---

## 3. Architecture & Design Decisions

### Why FastAPI?
- Async-first — critical for LLM API calls that can take 10–30 seconds
- Auto-generates Swagger UI at `/docs` for free
- Native Pydantic integration for type-safe request/response validation

### Why LangGraph?
LangGraph allows defining the agent pipeline as a **stateful graph** where each node is an agent. This makes it easy to:
- Add/remove agents without rewriting orchestration logic
- Pass state (outputs of previous agents) to subsequent agents
- Route between "design pipeline" and "simple chat" based on user intent

### Why PostgreSQL for conversation history?
The OpenAI API is **stateless** — it has no memory between calls. Every request must include the full conversation history. PostgreSQL stores this per `session_id` so users can return to previous conversations.

### Two API endpoints — why?
| Endpoint | When to use |
|----------|-------------|
| `POST /api/chat` | Free-form questions ("What is JWT?", "Explain microservices") |
| `POST /api/chat/design` | Structured form submission → triggers full 4-agent pipeline |

The structured form ensures the orchestrator receives all necessary information (team size, scale, deadline, constraints) to make accurate architectural decisions.

---

## 4. Project Structure

```
SystemDesignerAssistant/
├── backend/
│   ├── .env.example            ← copy to .env and fill in values
│   ├── .env                    ← your actual secrets (never commit this)
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── main.py                 ← start here
│   │
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── orchestrator.py     ← LangGraph pipeline
│   │   ├── requirements_agent.py
│   │   ├── architecture_agent.py
│   │   ├── database_agent.py
│   │   ├── api_agent.py
│   │   └── mock_responses.py
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   └── schemas.py          ← all Pydantic models
│   │
│   ├── db/
│   │   ├── __init__.py
│   │   ├── database.py         ← SQLAlchemy engine + ORM
│   │   └── queries.py          ← DB operations
│   │
│   └── api/
│       ├── __init__.py
│       └── chat_router.py      ← FastAPI routes
│
├── frontend/                   ← React app (in progress)
│   ├── Dockerfile
│   ├── package.json
│   └── src/
│       ├── App.js
│       ├── index.js
│       ├── hooks/useChat.js
│       ├── components/
│       │   ├── MessageBubble.js
│       │   └── ChatInput.js
│       └── pages/MainPage.js
│
└── docker-compose.yml          ← runs all 3 services together
```

---

## 5. Agent Pipeline

```
User submits form
        ↓
POST /api/chat/design
        ↓
build_prompt_from_form()   ← structures the form into a detailed prompt
        ↓
run_orchestrator()
        ↓
┌─────────────────────────────────────────────┐
│              LangGraph Graph                │
│                                             │
│  [Router Node]                              │
│   ↓ needs_design=true                       │
│                                             │
│  [Agent 1: Requirements]                    │
│   → functional_requirements                 │
│   → non_functional_requirements             │
│   → constraints, scale_estimation           │
│        ↓                                    │
│  [Agent 2: Architecture]                    │
│   → architecture_style (Monolith/Micro)     │
│   → services, tradeoffs, tech_stack         │
│        ↓                                    │
│  [Agent 3: Database]                        │
│   → entities, relationships                 │
│   → mysql_schema_sql, erd_mermaid           │
│        ↓                                    │
│  [Agent 4: API Designer]                    │
│   → endpoints, spring_security_config       │
│   → api_mermaid_diagram                     │
│   → docker_compose_snippet                  │
│        ↓                                    │
│  [Summarizer Node]                          │
│   → human-readable markdown summary         │
└─────────────────────────────────────────────┘
        ↓
ChatResponse {
  answer: "## System Design Complete...",
  structured_output: { requirements, architecture, database, api_design }
}
```

Each agent receives the **outputs of all previous agents** as context, ensuring decisions are consistent and coherent across the pipeline.

---

## 6. API Reference

### `GET /health`
Health check.
```json
{ "status": "ok", "service": "System Designer Assistant" }
```

### `POST /api/chat`
Free-form chat with the assistant.
```json
// Request
{
  "session_id": "unique-session-id",
  "message": "What is the difference between JWT and sessions?"
}

// Response
{
  "session_id": "unique-session-id",
  "answer": "JWT is stateless...",
  "structured_output": null
}
```

### `POST /api/chat/design`
Structured form → full 4-agent pipeline.
```json
// Request
{
  "session_id": "unique-session-id",
  "project_description": "An e-commerce platform with user auth and product catalog",
  "team_size": "2–3 developers",
  "scale": "1k–10k users/day",
  "deadline": "3–6 months",
  "tech_constraints": "Spring Boot, MySQL",
  "capital_constraints": "Free tier only",
  "extra_details": ""
}

// Response
{
  "session_id": "unique-session-id",
  "answer": "## System Design Complete...",
  "structured_output": {
    "requirements": { ... },
    "architecture": { ... },
    "database": { ... },
    "api_design": { ... },
    "created_at": "2026-03-08T10:55:06Z"
  }
}
```

### `GET /api/chat/history/{session_id}`
Get full conversation history for a session.

### `GET /api/chat/sessions`
List all session IDs.

### `DELETE /api/chat/history/{session_id}`
Clear history for a session.

---

## 7. Setup & Run Guide

### Prerequisites

| Tool | Version | Download |
|------|---------|----------|
| Python | 3.11+ (recommended) or 3.14 | https://python.org |
| PostgreSQL | 15+ | https://postgresql.org |
| Node.js | 18+ | https://nodejs.org |
| Git | any | https://git-scm.com |

> **Note:** The project was developed on Python 3.14. If you use Python 3.11, all packages install cleanly with pinned versions. On 3.14, some packages may need version adjustments.

---

### Step 1 — Clone the repository

```bash
git clone https://github.com/your-username/SystemDesignerAssistant.git
cd SystemDesignerAssistant
```

---

### Step 2 — Set up PostgreSQL

Open **SQL Shell (psql)** or **pgAdmin** and run:

```sql
CREATE USER designer_user WITH PASSWORD 'designer_pass';
CREATE DATABASE system_designer_db OWNER designer_user;
```

> You can change the credentials — just make sure they match your `.env` file.

---

### Step 3 — Set up Python virtual environment

```bash
cd backend

# Create virtual environment
python -m venv .venv

# Activate (Windows)
.venv\Scripts\activate

# Activate (Mac/Linux)
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

---

### Step 4 — Create your `.env` file

Copy the example:
```bash
cp .env.example .env
```

Edit `.env` with your values:
```env
OPENAI_API_KEY=sk-your-key-here
OPENAI_MODEL=gpt-4o
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_USER=designer_user
POSTGRES_PASSWORD=designer_pass
POSTGRES_DB=system_designer_db
APP_ENV=development
USE_MOCK=true
```

> Set `USE_MOCK=true` if you don't have an OpenAI API key yet. Set to `false` when you do.

---

### Step 5 — Run the backend

```bash
uvicorn main:app --reload
```

Expected output:
```
INFO:     Uvicorn running on http://127.0.0.1:8000
INFO:     Application startup complete.
✅ Database initialized
```

---

### Step 6 — Test the API

Open your browser at:
```
http://127.0.0.1:8000/docs
```

Try `POST /api/chat/design` with:
```json
{
  "session_id": "test-001",
  "project_description": "A banking app with accounts, transfers and auth",
  "team_size": "2–3 developers",
  "scale": "1k–10k users/day",
  "deadline": "3–6 months",
  "tech_constraints": "Spring Boot, MySQL",
  "capital_constraints": "Free tier only",
  "extra_details": ""
}
```

---

### Step 7 — Run the frontend (optional)

```bash
cd ../frontend
npm install
npm start
```

Frontend will be available at `http://localhost:3000`.

> Make sure the backend is already running before starting the frontend.

---

## 8. Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `OPENAI_API_KEY` | Yes* | — | Your OpenAI API key. *Not needed if USE_MOCK=true |
| `OPENAI_MODEL` | No | `gpt-4o` | OpenAI model to use |
| `POSTGRES_HOST` | Yes | `localhost` | PostgreSQL host |
| `POSTGRES_PORT` | No | `5432` | PostgreSQL port |
| `POSTGRES_USER` | Yes | — | PostgreSQL username |
| `POSTGRES_PASSWORD` | Yes | — | PostgreSQL password |
| `POSTGRES_DB` | Yes | — | PostgreSQL database name |
| `APP_ENV` | No | `development` | Environment name |
| `USE_MOCK` | No | `false` | `true` = use mock responses, no API key needed |

---

## 9. Mock Mode vs Real Mode

### Mock Mode (`USE_MOCK=true`)
- No OpenAI API key required
- Returns hardcoded responses from `agents/mock_responses.py`
- All 4 agents always return the same e-commerce example
- Useful for: frontend development, testing, demos

### Real Mode (`USE_MOCK=false`)
- Requires a valid `OPENAI_API_KEY`
- Each request calls GPT-4o with structured prompts
- The router agent automatically decides whether to run the full pipeline or answer conversationally
- Each agent receives the outputs of previous agents for context

### Switching modes
Simply change `USE_MOCK` in your `.env` file and restart the server:
```bash
# No restart needed if using --reload, just save .env
uvicorn main:app --reload
```

---

## 10. Docker Deployment

To run all 3 services (backend, frontend, PostgreSQL) with a single command:

```bash
# From the project root
docker-compose up --build
```

Services:
| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:8000 |
| Backend Docs | http://localhost:8000/docs |
| PostgreSQL | localhost:5432 |

To stop:
```bash
docker-compose down
```

To stop and delete all data:
```bash
docker-compose down -v
```

> Before running Docker, rename `.env.example` to `.env` and fill in your values. The `docker-compose.yml` reads from `backend/.env`.

---

## 11. Troubleshooting

### `Connect call failed ('127.0.0.1', 5432)`
PostgreSQL is not running.
- **Local install:** Start the PostgreSQL service from Windows Services or run `pg_ctl start`
- **Docker:** Run `docker-compose up db` first

### `password authentication failed for user "..."`
Wrong credentials in `.env`. Double-check `POSTGRES_USER` and `POSTGRES_PASSWORD` match what you created in psql.

### `ModuleNotFoundError: No module named 'langchain'`
Virtual environment is not activated or dependencies not installed.
```bash
.venv\Scripts\activate    # Windows
pip install -r requirements.txt
```

### `UserWarning: Core Pydantic V1 functionality isn't compatible with Python 3.14`
This is a warning, not an error — the app still runs. It means some LangChain internals use the older Pydantic v1 API. It will be fixed in future LangChain versions.

### OpenAI API errors (after setting `USE_MOCK=false`)
- Verify your `OPENAI_API_KEY` is correct and has credits
- Check you're using a supported model (`gpt-4o`, `gpt-4-turbo`)
- OpenAI rate limits: if you get 429 errors, wait a minute and retry

---

*Documentation generated for System Designer Assistant v1.0.0 — March 2026*
