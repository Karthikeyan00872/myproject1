# AI Question Generator (Flask + Gemini)

Production-ready local setup for generating question papers and dynamic topic-based questions using Google Gemini.

## What was optimized

- Fixed API contract mismatch between frontend (`multipart/form-data`) and backend (`JSON`) in question paper generation.
- Added robust request parsing and input validation for dynamic generation workflows.
- Added a dedicated `/api/generate-questions` endpoint for **topic + instruction-driven** Gemini question generation.
- Upgraded dependencies and pinned versions for reproducible installs.
- Added MongoDB fallback to built-in in-memory storage for smooth local startup when Mongo is not running.
- Improved status endpoints to expose Gemini model/runtime details and DB backend mode.

## Quick start

### 1) Create and activate a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
```

### 2) Install dependencies

```bash
pip install -r requirements.txt
```

### 3) Configure environment

```bash
cp .env.example .env
```

Set at least:

- `JWT_SECRET`
- `GEMINI_API_KEY` (or `GOOGLE_API_KEY` when using a Google AI Studio key)

Gemini note:
- If your configured `GEMINI_MODEL` is unavailable for your free-tier account, the backend automatically retries with compatible fallback models.
- The backend uses the new SDK format: `from google import genai` with `genai.Client(api_key=...)`.

Mongo options:
- Use local MongoDB via `MONGO_URI`
- Or keep `MONGO_FALLBACK_TO_MOCK=true` for local in-memory DB fallback (no extra packages required)

### 4) Run the app

```bash
python app.py
```

App runs on `http://localhost:5000`.

## Gemini integration details

### Dynamic Question Generation

Endpoint: `POST /api/generate-questions` (auth required)

Request JSON:

```json
{
  "subject": "Computer Science",
  "topics": "Graphs, BFS, DFS",
  "instructions": "Focus on problem-solving and include one tricky conceptual question",
  "difficulty": "mixed",
  "count": 6,
  "question_types": ["MCQ", "Short Answer", "Long Answer"]
}
```

Response contains generated content from Gemini and metadata.

### Question Paper Generation

Endpoint: `POST /api/generate-paper` (auth required)

Supports both:
- `application/json`
- `multipart/form-data` (used by current frontend)

If a `context_file` is uploaded, the backend extracts text and includes an excerpt in generation context.

## Health checks

- `GET /api/health` – includes `db_backend` (`mongodb` or `in-memory`)
- `GET /api/ai-status` – reports Gemini config/model availability

## Suggested production environment

- Set `MONGO_FALLBACK_TO_MOCK=false`
- Use strong `JWT_SECRET`
- Run with gunicorn:

```bash
gunicorn -w 2 -b 0.0.0.0:5000 app:app
```
