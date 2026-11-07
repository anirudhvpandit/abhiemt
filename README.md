# AbhiEMT — Engineering Operations Intelligence System

An integrated Flask monolith for engineering performance, delivery governance,
workforce capacity, quality/reliability and financial oversight — with an
AI intelligence layer grounded in verified database metrics.

## Quick start

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python run.py
```

Open http://localhost:5000 and sign in with a demo account:

| Username | Role | Password |
|---|---|---|
| `admin` | System Administrator | `abhiemt123` |
| `cto` | CTO / Executive | `abhiemt123` |
| `engmanager` | Engineering Manager | `abhiemt123` |
| `pm` | Project Manager | `abhiemt123` |
| `finance` | Finance / PMO Viewer | `abhiemt123` |

Demo data (5 departments, 8 teams, 30 employees, 12 projects, 220+ tasks,
sprints, deployments, defects, incidents, budgets, RAID, allocations) is
seeded automatically on first run. Admins can reset it via the sidebar
**Reset Demo Data** button.

## Modules

- **Executive Command Centre** — 12 KPIs, portfolio health, delivery/budget/defect/deploy charts, AI management summary
- **Engineering Productivity** — velocity, cycle time, WIP, bottlenecks, workload heatmap
- **Software Delivery Intelligence** — DORA four keys + release management, schedule variance
- **Workforce & Capacity** — utilisation formulas, over/under-allocation flags, skills inventory
- **Quality & Reliability** — defect metrics, test execution, incidents, MTTR/MTBF, uptime, scorecard
- **Financial Governance** — budgets, EAC, variance, stage gates, RAID, change requests
- **AbhiEMT Intelligence** — conversational AI with deterministic fallback; all metrics computed in Python
- **Reports** — HTML reports per period/project + Excel export
- **Excel Data Engine** — import (10 datasets), validation, error reports, templates, formatted exports

## Architecture

Modular monolith: `app/models` (SQLAlchemy), `app/services` (all business
logic/KPI math), `app/routes` (Jinja pages + `/api/v1` REST), `app/templates`.
SQLite by default; PostgreSQL-compatible via `DATABASE_URL`.

## AI layer

Set `AI_PROVIDER=openai` (or `anthropic`) and `AI_API_KEY` for LLM-narrated
answers. Without a key, a deterministic fallback analyses the same verified
metrics — the app never depends on an LLM for core functionality. The LLM
never calculates figures; all math happens in the service layer first.

## Tests

```bash
pytest
```

## Docker

```bash
docker compose up --build
```

## Deploy to Vercel

AbhiEMT runs on Vercel via `main.py` at the repo root, which exposes the Flask
`app` instance. Vercel's Python runtime detects Flask automatically and routes
all traffic to it — no `vercel.json` rewrites needed. The module also runs
`create_all()` and seeds demo data on cold start (only when `users` is empty).

### Steps

1. Push this repo to GitHub (or deploy the folder directly with `vercel --prod`).
2. In Vercel: **Add New → Project → Import** the repo. Framework preset: *Flask*.
3. Set environment variables (Project → Settings → Environment Variables):

   | Variable | Value |
   |---|---|
   | `SECRET_KEY` | long random string |
   | `DATABASE_URL` | `postgresql+psycopg2://user:pass@host/db` (Neon/Supabase/Vercel Postgres) — recommended |
   | `AI_PROVIDER` | `openai` or `anthropic` (optional) |
   | `AI_API_KEY` | your key (optional) |
   | `AI_MODEL` | e.g. `gpt-4o-mini` |

4. Deploy. On first cold start the function runs `create_all()` and seeds the
   demo dataset automatically (only when the `users` table is empty).

### Notes & limitations

- **Database**: serverless filesystems are ephemeral. Without `DATABASE_URL`
  the app falls back to SQLite in `/tmp` — data resets on cold starts. Use a
  hosted Postgres for anything persistent.
- **Upload size**: Vercel serverless bodies cap at ~4.5 MB, so Excel imports
  are limited to small files in production.
- **Sessions/limiter**: in-memory — fine for demos; use Redis-backed stores for
  multi-instance production.
- Alternatively run it anywhere with `docker compose up` or a plain
  `python run.py` behind gunicorn.

## API

Versioned JSON API under `/api/v1` — see `app/routes/api.py`. All endpoints
require login; finance endpoints enforce role checks.
