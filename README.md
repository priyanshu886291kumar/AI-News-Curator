# AI News Curator

An intelligent news aggregator that fetches headlines, **clusters similar stories** from multiple sources, and lets you **summarize** and **translate** any article using Google Gemini.

- **Backend:** FastAPI + LangChain (ReAct agents) + Google Gemini 1.5 Flash
- **Frontend:** React 19 (Create React App)
- **ML:** `sentence-transformers` (all-MiniLM-L6-v2) for semantic clustering of related articles

---

## Features

- Fetch top headlines by category (general, business, entertainment, health, science, sports, technology) via NewsAPI.
- Cluster near-duplicate articles from different outlets using sentence embeddings (cosine similarity > 0.8).
- Resolve source logos via Company Enrich API, with Clearbit and placeholder fallbacks.
- Summarize any article (scraped with `newspaper3k`).
- Translate summaries into Spanish, French, German, Hindi, or Chinese via the Google Translate API.

---

## Project structure

```
.
├── main.py              # FastAPI app & API endpoints (entry point)
├── agent.py             # LangChain agent for summarize / translate
├── get_news_agent.py    # LangChain agent for fetching & clustering news
├── tools.py             # Article extraction, summarization, translation tools
├── requirements.txt     # Python dependencies
├── vercel.json          # Vercel deployment config
└── news-frontend/       # React frontend
```

> Note: the `model_cache/` folder (the downloaded MiniLM model, ~86 MB) and `__pycache__/` are intentionally **not** committed. The model is downloaded automatically on first run.

---

## Prerequisites

- Python 3.10+
- Node.js 18+
- API keys (see [Environment variables](#environment-variables))

---

## Setup

### 1. Backend

```bash
# from the project root
python -m venv venv
# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt
```

Create a `.env` file in the project root (copy from `.env.example`) and fill in your keys.

Run the API:

```bash
uvicorn main:app --reload --port 8000
```

The API will be available at `http://localhost:8000`.

### 2. Frontend

```bash
cd news-frontend
npm install
npm start
```

The app runs at `http://localhost:3000` and talks to the backend at `http://localhost:8000`.

---

## Environment variables

Create a `.env` file in the project root:

| Variable | Used for | Where to get it |
|----------|----------|-----------------|
| `GOOGLE_API_KEY` | Gemini LLM (summarize/translate agents) | https://aistudio.google.com/app/apikey |
| `NEWS_API_KEY` | Fetching headlines | https://newsapi.org |
| `COMPANY_ENRICH_API_KEY` | Source logos | https://companyenrich.com |
| `TRANSLATE_API` | Google Translate API | https://console.cloud.google.com (Cloud Translation API) |

---

## API endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET`  | `/` | Health check |
| `GET`  | `/news?category=technology` | Fetch & cluster news for a category |
| `POST` | `/query` | Summarize / translate. Body: `{ "input": "Summarize this article: <url>" }` |

---

## Deployment

The backend includes a `vercel.json` for deployment to Vercel using the `@vercel/python` runtime. Note that on cold starts the transformer model download/clustering may exceed short serverless time limits — increase `maxDuration` if needed.
