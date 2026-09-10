---
name: mysql-insert
description: >-
  Generate or run safe local Docker MySQL seed SQL (DBeaver-ready), and ensure
  local Docker Redis is available for Playwright API tests. Use when seeding
  fixtures, mysql-insert, DBeaver SQL, or preparing data for Playwright local
  API tests. Prefer short verify JSON for downstream tools. Never use staging
  Cloud SQL or staging Redis.
---

# MySQL insert

## Token rules (do these first)

1. **Reuse** prior seed IDs/paths under `/tmp/*-seed.json` when still valid — do not re-discover the whole schema.
2. **Do not** dump `.env` / passwords into chat or reports.
3. **One** discovery round → insert → verify; stop on success.
4. **Local Docker only** — MySQL (`fithub-mysql-local`) and Redis (`fithub-redis-local`). Never Cloud SQL proxy, staging DB/Redis, `fit-hub-staging`, or port **3307**.
5. Long SQL / Docker recipes → [reference.md](reference.md) only when needed.

## When to use / skip

| Use | Skip |
|-----|------|
| Seed / INSERT / DBeaver SQL against **local Docker MySQL** | Staging / production unless user explicitly asks |
| Prep data for Playwright local API (MySQL seed; Redis must be up) | PR create unless PR adds SQL seeds |
| Start / verify local Docker MySQL + Redis | Firestore / Dynamo setup |

## Checklist

```
- [ ] 1. Docker daemon running (Docker Desktop or Colima)
- [ ] 2. Docker MySQL up on 127.0.0.1:3306 (fithub-mysql-local)
- [ ] 3. Docker Redis up on 127.0.0.1:6379 (fithub-redis-local) — required when pairing with Playwright
- [ ] 4. Target table + required columns (ask only if missing)
- [ ] 5. Discovery SELECT for FKs by name (no invented UUIDs)
- [ ] 6. INSERT (prefer INSERT…SELECT) + verify SELECT
- [ ] 7. Write seed JSON to /tmp/<case>-seed.json (no secrets)
- [ ] 8. Cleanup DELETE only if user asks
```

## Local Docker stack (required)

Before seeding or Playwright:

| Container | Port | Purpose |
|-----------|------|---------|
| `fithub-mysql-local` | `3306` | Seed + API MySQL |
| `fithub-redis-local` | `6379` | API cache (no password) |

Start commands and env alignment: [reference.md](reference.md).

Match `.env.testing` in **BACKEND-GOLANG**, **free-trial-service**, and **lms-service** (already pointed at this stack).

## Execute against local Docker MySQL

When the user asks to **run** seeds (not just print SQL):

| Step | Rule |
|------|------|
| DB | **Docker MySQL only** — `fithub-mysql-local` |
| Host/port | `127.0.0.1:3306`. Never `3307` / cloud-sql-proxy |
| Creds | `fithub` / `fithub_local` / `fithub-local` (match Docker + `.env.testing`) |
| Env | Parse `.env.testing` in Python — never `source` in zsh; never use Cloud SQL instance name |
| Client | Homebrew `mysql` may fail auth plugin → `/tmp/.../venv` + **pymysql** |
| Schema | If DB is empty, apply schema + FK lookup rows before INSERT |
| Redis | Confirm `redis-cli ping` → `PONG` when work continues to Playwright |
| Output | Print verify row + path to seed JSON only |

### Forbidden (unless user explicitly overrides)

- `~/bin/cloud-sql-proxy`
- `fit-hub-staging:asia-southeast1:fithub-db-staging`
- Staging / production MySQL or Redis hosts
- Seeding against remote staging IPs

## `leads` free-trial recurring seed (common)

Required: `id`, **`parent_id` (= id)**, `club_id`, `phone`, `email`, `name`, `status_id`, `source_id`, `preferred_contact_time_id`, timestamps, `created_by`/`updated_by`.  
Set `deal_at = NULL` to exercise empty Dynamo `dealDate`.  
Mark `remarks` like `seed:<ticket-or-case>`.

FK gotchas (local Docker — same names as prod-like fixtures):

- Club: prefer **`FIT HUB BENHIL`** (ensure that club row exists locally).
- Contact time name is **`Anytime`** (not `ANYTIME`).
- Status table is **`leads_status`** (not `lead_status`).

## Output shape (chat)

1. One line: what was seeded  
2. Seed JSON path + key fields (`id`, `phone`, `club`)  
3. Cleanup SQL (only if useful)

Details: [reference.md](reference.md).
