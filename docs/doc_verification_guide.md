# PHL — Verification Guide

## Purpose

Use this document to choose the right verification path before and after any change in a PHL project.

This is not a catalog of every command. It is a curated guide for:
- fast local checks,
- API contract validation,
- deployment verification,
- debugging live issues on VPS.

Use the cheapest verification that can detect the risk of the change.

---

## Verification Strategy

Prefer this order:

1. Unit tests for pure logic changes.
2. API contract tests for endpoint behavior.
3. Manual smoke test against staging/VPS for deployment or config changes.
4. Live VPS inspection only when production behavior diverges from expected.

Do not deploy to production without at least running the API smoke check.

---

## Quick Map

### I changed pure logic (pipeline, parsing, email formatting)

Run unit tests:
```bash
python -m pytest tests/unit -q
```

### I changed an API endpoint or its response shape

1. Run integration tests:
```bash
python -m pytest tests/integration -q
```

2. Update the contract file in `phlara/apis` — e.g. `urgara-newsletter.md`.

3. Smoke test the endpoint locally:
```bash
curl -s -H "X-API-Key: <key>" http://localhost:5000/api/v1/status | python -m json.tool
```

### I changed the Docker setup or Nginx config

1. Build and run locally:
```bash
docker compose up --build
```

2. Verify the health endpoint responds:
```bash
curl -s http://localhost:5000/api/v1/healthz
```

3. Check Nginx config syntax:
```bash
nginx -t
```

### I changed tenant config (`tenants/<slug>/config.yaml`)

1. Start the project locally with that tenant active.
2. Verify `/status` returns the expected branding and recipient count.
3. Do NOT send real emails during local testing — check the `.env` has `EMAIL_DRY_RUN=true`.

---

## VPS Verification

### Check that the service is alive

```bash
curl -s https://urgara.sytes.net/api/v1/healthz
```

Expected:
```json
{ "status": "ok", "tenant": "urgara" }
```

### Check the current pipeline state

```bash
curl -s -H "X-API-Key: <key>" https://urgara.sytes.net/api/v1/status | python -m json.tool
```

### SSH into the VPS to inspect logs

```bash
ssh root@157.230.182.52
docker logs <container_name> --tail 100
```

### Restart the container after a failed deploy

```bash
ssh root@157.230.182.52
cd /app
docker compose down && docker compose up -d
```

---

## API Smoke Checks

### List newsletters

```bash
curl -s -H "X-API-Key: <key>" https://urgara.sytes.net/api/v1/newsletters | python -m json.tool
```

### Get today's newsletter detail

```bash
curl -s -H "X-API-Key: <key>" https://urgara.sytes.net/api/v1/newsletters/$(date +%Y%m%d) | python -m json.tool
```

### Get email open events

```bash
curl -s -H "X-API-Key: <key>" https://urgara.sytes.net/api/v1/newsletters/$(date +%Y%m%d)/events | python -m json.tool
```

---

## Anti-Patterns

Do not:
- Push to `main` without verifying locally first — it deploys immediately.
- Test email sending in production unless explicitly approved by phlara.
- Modify `.env` on the VPS without documenting the change.
- Trust only the admin web UI — always cross-check with the API.

---

## Contract Update Checklist

When any API endpoint changes, before merging:

- [ ] API tests pass (`pytest tests/integration`)
- [ ] Updated the contract file in `phlara/apis`
- [ ] Smoke tested the endpoint locally
- [ ] Verified `/healthz` still responds after deploy
- [ ] Notified dashboard consumers if the response shape changed
