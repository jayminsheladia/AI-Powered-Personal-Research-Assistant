# AI-Powered Personal Research Assistant

Upload research papers, get structured summaries, chat with a paper (RAG), find related work, compare papers, take linked notes, browse a knowledge graph, and get literature-review support.

## Stack
- **Backend**: FastAPI, SQLAlchemy 2.0 + Alembic, Postgres + pgvector
- **LLM**: Google Gemini (summaries, structured extraction, RAG, comparisons) — free tier, no card required
- **Embeddings**: Voyage AI
- **External data**: Semantic Scholar API (citations, references, recommendations)
- **Frontend**: Next.js (App Router) + TypeScript + Tailwind

## Setup

1. Copy `.env.example` to `.env` and fill in:
   - `GEMINI_API_KEY` — required, get one free at [aistudio.google.com](https://aistudio.google.com)
   - `VOYAGE_API_KEY` — required, get one free (200M tokens) at [voyageai.com](https://voyageai.com)
   - `JWT_SECRET` — any random string
   - `SEMANTIC_SCHOLAR_API_KEY` — optional, works unauthenticated at lower rate limits

2. Start everything:
   ```bash
   docker-compose up --build
   ```
   This starts Postgres (with pgvector), runs migrations, and starts the backend (`:8000`) and frontend (`:3000`).

3. Open http://localhost:3000, sign up, and upload a PDF.

## Running without Docker

Backend:
```bash
cd backend
python3 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
# requires a Postgres with the pgvector extension available.
# If you started `docker-compose up db` separately, it's published on localhost:5433
# (not 5432, to avoid clashing with any Postgres already running on your machine).
export DATABASE_URL="postgresql+psycopg://research_assistant:research_assistant@localhost:5433/research_assistant"
alembic upgrade head
uvicorn app.main:app --reload
```

Backend tests (no external services required — LLM/embedding calls are mocked):
```bash
cd backend && source .venv/bin/activate && pytest
```

Frontend:
```bash
cd frontend
npm install
npm run dev
```

## Project layout
See `backend/app/` for services (PDF extraction, chunking, embeddings, RAG, metadata extraction, summarization, comparison, citation formatting, knowledge graph, literature review support) and API routes. See `frontend/app/` for pages (dashboard, upload, documents, chat, related papers, notes, compare, graph, review, search).

## Notes on scope
This is a full scaffold covering every module in the spec with working first-pass logic, not a polished production app. In particular:
- Auth is basic email/password + JWT, no password reset/email verification.
- PDF section detection is heuristic (regex over common academic headers) — it won't be perfect on unusual paper layouts.
- The knowledge graph draws document–document edges from shared tags and cached citation relations rather than a heavier clustering algorithm.
- Ingestion runs as a FastAPI background task; for heavier load, swap in a real task queue (Celery/RQ).
