# PHL Ecosystem — Architecture Guide

## Purpose

This document is the canonical architecture reference for AI agents working on any PHL project.

Use it to understand:
1. How the projects relate to each other
2. What each layer owns
3. Where a new file, endpoint, or feature belongs
4. What contracts and boundaries must not change without updating `phlara/apis`

---

## Ecosystem Map

```
urgara-newsletter  ──┐
                     ├──► Dashboard Nivel 1 (Cristian) ──► Dashboard PHL (nivel dios)
pds-newsletter     ──┘
```

All projects communicate via REST API. Authentication: `X-API-Key` header.  
API contracts are the source of truth and live in **`phlara/apis`** (repo companion).

---

## Projects

### urgara-newsletter
- **What:** Daily newsletter engine for URGARA (grain receivers union)
- **Repo:** `github.com/phlara/urgara-newsletter` (private)
- **VPS:** `157.230.182.52` / `urgara.sytes.net` — DigitalOcean, Docker
- **Stack:** Python, Docker, Nginx, Let's Encrypt, Brevo (email delivery)
- **Deploy:** GitHub Actions — push to `main` → deploy
- **Admin web:** `https://urgara.sytes.net` (internal)
- **Public API:** `https://urgara.sytes.net/api/v1/`
- **Contract:** `phlara/apis/urgara-newsletter.md`

### pds-newsletter
- **What:** Same newsletter engine, tenant Puerto Dock Sud (PDS)
- **Repo:** `github.com/phlara/Puerto-Dock-Sud` (private)
- **Stack:** Same as urgara-newsletter
- **Contract:** `phlara/apis/pds-newsletter.md`

### Dashboard Nivel 1 (Cristian)
- **What:** External dashboard that consumes data from both newsletter APIs
- **Stack:** React
- **Visibility:** Cristian's operational view — reads pipeline status, sends, opens
- **Contract:** `phlara/apis/dashboard-nivel1.md`

### Dashboard PHL (nivel dios)
- **What:** phlara's unified view of all projects in the ecosystem
- **Stack:** React
- **Status:** In construction
- **Visibility:** Full — all projects, all metrics, all alerts
- **Contract:** `phlara/apis/dashboard-phlara.md`

---

## API Contract Rules

1. Every API change in any project must be reflected in `phlara/apis`.
2. Dashboards read `phlara/apis` to know what changed.
3. Never break a published endpoint without versioning it first.
4. All endpoints require `X-API-Key` except `/healthz` and tracking pixels.

### Standard endpoints every newsletter project exposes

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/healthz` | GET | Liveness check — no auth |
| `/status` | GET | Current pipeline state, email stats |
| `/newsletters` | GET | List of all generated newsletters |
| `/newsletters/<date_str>` | GET | Detail for a specific date (`YYYYMMDD`) |
| `/newsletters/<date_str>/html` | GET | HTML of the newsletter |
| `/newsletters/<date_str>/events` | GET | Email open events |
| `/newsletters/<date_str>/approve` | POST | Approve for sending |
| `/newsletters/<date_str>/send-email` | POST | Send via Brevo |

### Planned (in development)

| Endpoint | Description |
|----------|-------------|
| `GET /alerts` | Operator tickets / operational alerts |
| `POST /alerts` | Create a ticket |
| `POST /alerts/<id>/resolve` | Resolve a ticket |

---

## Newsletter Project Layout

Each newsletter project follows this structure:

```
.
├── tenants/<slug>/
│   └── config.yaml        # Tenant-specific config (branding, recipients, etc.)
├── src/
│   ├── pipeline/          # Daily content generation logic
│   ├── email/             # Brevo integration, send logic
│   ├── api/               # Flask/FastAPI routes
│   └── tracking/          # Email open pixel
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
DigitalOcean VPS
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
- API keys are generated with: `python3 -c "import secrets; print(secrets.token_urlsafe(32))"`

---

## What AI Agents Must Not Do

- Never commit `.env` or any file containing secrets.
- Never change a public API endpoint shape without updating `phlara/apis`.
- Never change tenant config files without understanding the multi-tenant model.
- Never force-push to `main` — it triggers a production deploy.

---

## Verification Reference

See `docs/doc_verification_guide.md` for concrete commands to validate changes.
See `docs/doc_ai_planning_mode.md` for the planning workflow before implementing anything.
