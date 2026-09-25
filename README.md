<div align="center">

# BharatSaarthi AI

**An AI-Powered Voice-First Scheme Navigator for Every Citizen**

`voice` `eligibility` `deterministic rules` `explainable AI` `zero-middlemen`

</div>

---

## Problem

Millions of eligible Indian citizens miss public benefits because scheme information lives
on **scattered portals**, in **bureaucratic language**, and behind **documentation
friction**. The information exists; finding the right scheme at the right time doesn't.

## Solution

BharatSaarthi is a voice-first assistant that listens to a citizen in Hindi, English,
Hinglish, Marathi or Tamil, reads their profile, and instantly shows which government
schemes they actually qualify for — **with the rules shown, not hand-waved**.

**The core promise: eligibility is decided by an open deterministic rules engine. The AI
only explains. It never guesses, never invents amounts, never sells anything.**

## Architecture

```
 voice/text/photo ──► 1. Extraction   (Gemini JSON schema · keyword fallback)
                         │
                         ▼
                       2. Profile      (state, age, income, occupation, category …)
                         │
                         ▼
   ┌──────────────────── 3. Ranking ──────────────────────────────────┐
   │  deterministic rules (fit)  +  state bonus  +  semantic match    │
   │  (text-embedding-004 vectors · numpy cosine · cached JSON)       │
   └──────────────────────────────────────────────────────────────────┘
                         │
                         ▼
                       4. Cards       (verdict · evidence · documents · .gov.in link)
                         │
                         ▼
                       5. Explain     (Gemini streamed via SSE · deterministic fallback · read-aloud)
```

**Fallback-first by construction** — every Gemini call degrades to a deterministic path,
so the demo never dies on a flaky network or an expired free-tier quota.

## Repo layout

```
backend/
  app/
    main.py                 FastAPI app (11 endpoints)
    config.py               env, origins, official-domain guard
    schemas.py              Pydantic contracts
    services/
      rules.py              deterministic eligibility rules
      ranking.py            state-aware + semantic blend
      readiness.py          document checklist & guidance
      gemini.py             extraction, explanation, streaming
      embeddings.py         cached text-embedding-004 + numpy cosine
      bhashini.py           Bhashini ULCA translation (optional)
      dataset.py            scheme loading, versioning, freshness
    tests/                  39 tests (rules, api, embeddings, features)
  scripts/
    build_embeddings.py     one-time vector cache builder
    verify_dataset.py       honesty checker for data/schemes.json
    fetch_schemes.py        BeautifulSoup portal scraper → candidates
frontend/
  src/                      Vite + React 19 + TS (strict), Web Speech I/O
data/
  schemes.json              16 verified schemes (versioned, dated)
docs/
  dataset-notes.md
```

## Run it

```bash
# 1. Backend
cd backend
python -m venv .venv                       # once
.venv/Scripts/python -m pip install -r requirements.txt
.venv/Scripts/python -m uvicorn app.main:app --port 8000

# 2. Frontend (new terminal)
cd frontend
npm install                                 # once
npm run dev                                 # → http://localhost:5173
```

No `GEMINI_API_KEY`? Deterministic keyword extraction and
fallback explanations take over automatically. With a key (from
[AI Studio](https://aistudio.google.com)) you get full Gemini extraction, streaming
explanations and read-aloud quality. (`DEMO_MODE=1` bypasses Gemini deliberately for reproducible demos.)

**One-time (after adding `GEMINI_API_KEY`):** run `backend/.venv/Scripts/python backend/scripts/build_embeddings.py` from the repo root to build `data/scheme_embeddings.json`; semantic ranking degrades gracefully without it.

## Environment

See `.env.example`. Key variables:

| Variable | Purpose |
|---|---|
| `GEMINI_API_KEY` | enables Gemini extraction / explanation / embeddings |
| `GEMINI_MODEL` | default `gemini-3.6-flash` |
| `GEMINI_EMBED_MODEL` | default `gemini-embedding-001` |
| `ALLOWED_ORIGINS` | CORS allow-list (comma separated) |
| `DEMO_MODE` | `1` pins the deterministic path for reproducible demos |
| `VITE_API_BASE` | frontend env — where the browser finds the API |

## Data honesty (read this before demo)

- `data/schemes.json` is **versioned** (`rule_version`) and **dated**
  (`last_verified_date`) — always re-verify before presenting.
- `backend/scripts/verify_dataset.py` checks every official URL and only stamps a new
  date on schemes whose portal actually responded.
- `backend/scripts/fetch_schemes.py` produces **candidates only**; nothing auto-merges.
  SPA-only portals (myscheme.gov.in, india.gov.in) are detected and skipped loudly.
- Scheme amounts change. When criteria change, **bump every touched record's
  `rule_version`**; the Data Status page and `/health` surface it.

## Voice (browser support)

Input + read-aloud use the **Web Speech API** (no SDK, no audio upload):
Chrome/Edge fully supported. Firefox/Safari degrade to text input + read-aloud off.

## Deployment

- **Frontend:** static build (`npm run build`) → Vercel/Netlify. Set `VITE_API_BASE`.
- **Backend:** `render.yaml` ships the Render blueprint; preloaded JSON means zero DB.
  Shrink-wrap local dev origins in `ALLOWED_ORIGINS` before opening CORS.

## How the product stays honest

| Risk | Approach |
|---|---|
| Live portals change | scrape once → static dataset, refreshed by maintainers, never auto-merged |
| Offline / weak connections | fully static dataset with offline fallback |
| Rules can change | versioned dataset with dated records and a visible "last verified" date |
| Wrong guesses | eligibility decided only by published official rules; explanations never invent amounts |

<div align="center">बिलकुल निःशुल्क सलाह · Eligibility decided by open rules — never guessed.</div>