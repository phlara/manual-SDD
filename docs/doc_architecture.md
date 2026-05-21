# PHL Ecosystem — Architecture Guide

## Purpose

This document is the canonical architecture reference for AI agents working on any PHL project.

Use it to understand:
1. How the projects relate to each other
2. What each layer owns and where new code belongs
3. What contracts and boundaries must not change without updating `phlara/apis`

---

## Ecosystem Map

```
urgara-newsletter  ──┐
                     ├──► Dashboard Nivel 1 (Cristian)
pds-newsletter     ──┘              │
                                    │ estado / healthcheck
urgara-newsletter  ──┐              │
pds-newsletter     ──┼──────────────┴──► Dashboard PHL (phlara)
Dashboard Nivel 1  ──┤
boostflow          ──┘
```

- **Dashboard Nivel 1** consumes: urgara-newsletter + pds-newsletter.
- **Dashboard PHL** consumes everything: urgara + PDS + Dashboard Nivel 1 + boostflow.

All projects communicate via REST API. Authentication: `X-API-Key` header.  
API contracts are the source of truth and live in **`phlara/apis`** (repo companion).

---

## Projects

### urgara-newsletter
- **What:** Daily newsletter engine for URGARA (grain receivers union)
- **Repo:** `github.com/phlara/urgara-newsletter` (private)
- **VPS:** `urgara.sytes.net` — DigitalOcean, Docker
- **Stack:** Python, Docker, Nginx, Brevo (email delivery)
- **Deploy:** GitHub Actions — push to `main` → deploy
- **Public API:** `https://urgara.sytes.net/api/v1/`
- **Contract:** `phlara/apis/urgara-newsletter.md`

### pds-newsletter
- **What:** Same newsletter engine, tenant Puerto Dock Sud (PDS)
- **Repo:** `github.com/phlara/Puerto-Dock-Sud` (private)
- **Stack:** Same as urgara-newsletter (Python, Docker, Nginx, Resend)
- **Public API:** `https://pds.intelia.today/api/v1/`
- **Contract:** `phlara/apis/pds-newsletter.md`

### Dashboard Nivel 1 (Cristian)
- **What:** Operational dashboard for Cristian — consumes both newsletter APIs
- **Repo:** `github.com/phlara/dash-cristian` (private)
- **Stack:** PHP + vanilla JS, hosted on Hostinger shared hosting
- **URL:** `https://artemis-tech.io/cristian/`
- **Contract:** `phlara/apis/dashboard-nivel1.md`

### BoostFlow
- **What:** SaaS de diagnóstico digital para PyMEs — GMB, SEO, redes sociales vía Apify + IA
- **Repo:** `github.com/phlara/mrp` (private)
- **Stack:** FastAPI (Render) + React/Vite (Render static) + PostgreSQL (Supabase)
- **URL:** `https://boostflow.digital` / API: `https://boostflow-api.onrender.com`
- **Contract:** `phlara/apis/boostflow.md`

### Dashboard PHL
- **What:** phlara's unified view of all projects in the ecosystem
- **Stack:** PHP + vanilla JS (same pattern as Dashboard Nivel 1)
- **Status:** In construction
- **URL:** `https://artemis-tech.io/control/`
- **Contract:** `phlara/apis/dashboard-phlara.md` (pending)

---

## Layer Responsibilities (Any Stack)

Regardless of the language or framework, every backend project must enforce these boundaries:

| Layer | Responsibility | Must NOT contain |
|-------|---------------|-----------------|
| **Presentation** — routes, controllers, views | Parse request → call service → return response | Business logic, DB queries |
| **Application** — services, use cases | Orchestrate: validate input, call domain + repositories | HTTP concerns, raw DB access |
| **Domain** — models, rules, entities | Business rules, invariants, pure transformations | HTTP, DB, or external service knowledge |
| **Infrastructure** — DB, external APIs, queues | Technical adapters: talk to DB, call external APIs | Business logic |

For frontend projects, enforce this equivalent separation:

| Layer | Responsibility |
|-------|---------------|
| **Views / Components** | Render UI, handle events, display data |
| **Service / API layer** | Centralized functions that call backend endpoints |
| **Config** | API base URLs, keys — always gitignored |

---

## File Placement (Decision Guide)

When adding code, decide by **intent**:

| Intent | Where it goes |
|--------|--------------|
| New HTTP endpoint | Presentation layer (routes / controllers) |
| New business rule or validation | Application or Domain layer |
| New DB query | Infrastructure / Repository layer |
| New external API call | Infrastructure / Integration layer |
| New UI page | views / pages folder |
| New reusable UI element | components folder |
| New API call from frontend | service / api layer (not inline) |
| Shared helper, no business meaning | utils / helpers |

**Never mix layers.** If a module needs HTTP context AND DB access, split it.

---

## Anti-Patterns — Never Introduce

- Business logic inside route handlers or controllers
- Direct DB queries inside routes
- Inline API calls in frontend views without a service layer
- Raw SDK/library exceptions propagated across layer boundaries
- The same responsibility duplicated in two different layers
- God functions or files that handle everything
- Secrets or API keys committed to the repo or hardcoded in frontend code

---

## API Contract Rules

1. Every API change in any project must be reflected in `phlara/apis` **in the same work cycle**.
2. Dashboards read `phlara/apis` to know what is available — a shipped endpoint without a contract does not exist for consumers.
3. Never break a published endpoint shape without updating the contract and notifying consumers.
4. All endpoints require `X-API-Key` except `/healthz` and tracking pixels.

### Standard endpoints every newsletter project exposes

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/healthz` | GET | Liveness check — no auth |
| `/status` | GET | Current pipeline state, email stats |
| `/newsletters` | GET | List of all generated newsletters |
| `/newsletters/<date_str>` | GET | Detail for a specific date (`YYYYMMDD`) |
| `/newsletters/<date_str>/html` | GET | Rendered HTML |
| `/newsletters/<date_str>/events` | GET | Email open events |
| `/newsletters/<date_str>/approve` | POST | Approve for sending |
| `/newsletters/<date_str>/send-email` | POST | Send via email provider |
| `/alerts` | GET | Operator tickets / operational alerts |
| `/alerts/<id>/resolve` | POST | Resolve a ticket |

---

## Newsletter Project Layout

```
.
├── newsletter_engine/
│   ├── pipeline/          # Daily content generation logic
│   ├── integrations/      # Email provider, external APIs
│   ├── admin/             # Flask admin routes and views
│   └── static/            # Frontend assets for admin
├── tenants/<slug>/
│   └── config.yaml        # Tenant-specific config (branding, recipients)
├── docker-compose.yml
├── Dockerfile
└── .env                   # Secrets — never committed
```

---

## Deployment Model

```
GitHub push to main
       │
       ▼
GitHub Actions (CI/CD)
       │
       ▼
VPS (DigitalOcean) or Render
       │
       ▼
Docker container
       │
       ▼
Nginx (reverse proxy + TLS)
       │
       ▼
Public HTTPS endpoint
```

- Secrets live in `.env` on the VPS — never in the repo.
- API keys: `python3 -c "import secrets; print(secrets.token_urlsafe(32))"`

---

## What AI Agents Must Not Do

- Never commit `.env` or any file containing secrets.
- Never change a public API endpoint shape without updating `phlara/apis`.
- Never force-push to `main` — it triggers a production deploy.
- Never assume the frontend or backend stack — read `CLAUDE.md` first.
- Never plan UI that consumes an endpoint not yet documented in `phlara/apis`.

---

## Verification Reference

See `docs/doc_verification_guide.md` for concrete commands to validate changes.  
See `docs/doc_ai_planning_mode.md` for the planning workflow before implementing anything.
