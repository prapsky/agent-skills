---
name: playwright-local-api-test
description: >-
  Run local Playwright API tests against Docker MySQL + Docker Redis, with seed
  data from mysql-insert, write HTML reports, and optionally post a short PR
  test comment. Use when testing locally with Playwright, verifying an API with
  seeded MySQL data, or reporting results. Never use staging MySQL or Redis.
---

# Playwright — local API test

## Token rules (do these first)

1. **Reuse** `BE - BACKEND-GOLANG/playwright/local-api/` (or the repo’s existing folder) — do not scaffold a new project.
2. **Reuse** `/tmp/<case>-seed.json` + `/tmp/<case>-body.json` from `mysql-insert`.
3. **Do not** curl the mutating endpoint and then run Playwright on the **same** seed phone without reseed (first call consumes recurring path; second hangs/times out).
4. **Local Docker MySQL + Redis only** — `fithub-mysql-local` + `fithub-redis-local`. Never Cloud SQL proxy, staging DB/Redis, `fit-hub-staging`, or port **3307**. See `mysql-insert` + [reference.md](reference.md).
5. Keep chat short: pass/fail + URLs + report paths. Details → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Local API test + HTML report | SQL-only → `mysql-insert` |
| PR comment with local proof (if user asks) | UI E2E unless asked |
| | PR create unless PR adds Playwright tests |

## Defaults (BACKEND-GOLANG free-trial)

| Item | Value |
|------|--------|
| Dir | `playwright/local-api/` |
| URL | `http://127.0.0.1:5005/v1/leads/free-trial` |
| MySQL | Docker `fithub-mysql-local` → `127.0.0.1:3306` / `fithub-local` / `fithub` |
| Redis | Docker `fithub-redis-local` → `127.0.0.1:6379`, empty password |
| Env | Load service `.env.testing` (already local Docker) via Python — never `source` in zsh |
| Start API | `FUNCTION_TARGET=…` `PORT=5005`; `go build -o tmp/main ./cmd/main.go && ./tmp/main` |
| Timeout | `test.setTimeout(180_000)` for free-trial (Redis/Firestore latency) |
| Specs | Table-driven; attach seed/request/response; write `/tmp/<case>-pw-result.json` |

### Preflight (before starting the API)

```bash
# MySQL
docker start fithub-mysql-local 2>/dev/null || true
docker exec fithub-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -pfithub_root_local --silent

# Redis
docker start fithub-redis-local 2>/dev/null || \
  docker run -d --name fithub-redis-local -p 6379:6379 redis:7-alpine
docker exec fithub-redis-local redis-cli ping   # PONG
```

Full `docker run` for MySQL: `mysql-insert` [reference.md](../mysql-insert/reference.md).

### Local connection rules (do not point at staging)

| Service | MySQL connection |
|---------|------------------|
| BACKEND-GOLANG | `DATABASE_INSTANCE_CONNECTION_TYPE=tcp` + `WHITELIST_DATABASE_PRIVATE_CONNECTION_TYPE=.*` + `DATABASE_HOSTNAME_PRIVATE=127.0.0.1` |
| free-trial-service | `DATABASE_INSTANCE_CONNECTION_TYPE=tcp` |
| lms-service | `DATABASE_MASTER_INSTANCE_CONNECTION_TYPE=private` (code uses this for `@tcp`) |

All three: `REDIS_HOST=127.0.0.1`, `REDIS_PORT=6379`, `REDIS_PASSWORD=` empty. Clear Cloud SQL instance connection names.

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. fithub-mysql-local healthy on 3306
- [ ] 3. fithub-redis-local healthy on 6379 (PONG)
- [ ] 4. Seed JSON ready (mysql-insert) + body mapping
- [ ] 5. API listening on local Docker MySQL + Redis (one probe OR Playwright — not both on same seed)
- [ ] 6. Run: npx playwright test tests/<spec>.ts --reporter=list,html
- [ ] 7. Write playwright-report/result.html
- [ ] 8. If user asked: post PR comment (format below)
```

## PR comment format (only when asked)

Use **multi-line** JSON (not one line):

1. **The issue** + issue log  
2. **Endpoint URL local**  
3. **Request body**  
4. **Response**  
5. **Explanation** — one simple sentence  

## Security

- No secrets in HTML, attachments, or PR comments  
- **Local Docker MySQL + Redis only** unless user explicitly asks otherwise  
- Never staging MySQL, staging Redis, Cloud SQL proxy, or production  

Templates & mapping: [reference.md](reference.md).
