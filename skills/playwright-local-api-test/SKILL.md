---
name: playwright-local-api-test
description: >-
  Run local Playwright API tests against Docker MySQL + Docker Redis, with seed
  data from mysql-insert, write HTML reports, and optionally post a short PR
  test comment. Use when testing locally with Playwright, verifying an API with
  seeded MySQL data, or reporting results. Never use remote staging MySQL or
  Redis.
---

# Playwright — local API test

## Token rules (do these first)

1. **Reuse** the repo’s existing `playwright/local-api/` (or equivalent) folder — do not scaffold a new project.
2. **Reuse** `/tmp/<case>-seed.json` + `/tmp/<case>-body.json` from `mysql-insert`.
3. **Do not** curl a mutating endpoint and then run Playwright on the **same** consumed seed without reseed (second call may hang/time out).
4. **Local Docker MySQL + Redis only** — `app-mysql-local` + `app-redis-local`. Never cloud DB proxies or remote staging DB/Redis. See `mysql-insert` + [reference.md](reference.md).
5. Keep chat short: pass/fail + URLs + report paths. Details → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Local API test + HTML report | SQL-only → `mysql-insert` |
| PR comment with local proof (if user asks) | UI E2E unless asked |
| | PR create unless PR adds Playwright tests |

## Defaults (adjust per project)

| Item | Typical value |
|------|----------------|
| Dir | `playwright/local-api/` (or the repo’s existing path) |
| URL | `http://127.0.0.1:<port>/<api-path>` |
| MySQL | Docker `app-mysql-local` → `127.0.0.1:3306` / match local env |
| Redis | Docker `app-redis-local` → `127.0.0.1:6379`, empty password |
| Env | Load local env file via Python — never `source` in zsh |
| Start API | Project’s usual local start (build + run / function target / compose) |
| Timeout | Raise per endpoint if cache / external deps are slow |
| Specs | Table-driven; attach seed/request/response; write `/tmp/<case>-pw-result.json` |

### Preflight (before starting the API)

```bash
# MySQL
docker start app-mysql-local 2>/dev/null || true
docker exec app-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -papp_root_local --silent

# Redis
docker start app-redis-local 2>/dev/null || \
  docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine
docker exec app-redis-local redis-cli ping   # PONG
```

Full `docker run` for MySQL: `mysql-insert` [reference.md](../mysql-insert/reference.md).

### Local connection rules (do not point at staging)

| Concern | Rule |
|---------|------|
| MySQL | Plain TCP to `127.0.0.1:3306` (disable cloud connectors for the local run) |
| Redis | `REDIS_HOST=127.0.0.1`, `REDIS_PORT=6379`, empty password unless you set one |
| Secrets | Do not print passwords; do not post them in HTML/PR comments |

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. app-mysql-local healthy on 3306
- [ ] 3. app-redis-local healthy on 6379 (PONG)
- [ ] 4. Seed JSON ready (mysql-insert) + body mapping
- [ ] 5. API listening on local Docker MySQL + Redis (one probe OR Playwright — not both on same consumed seed)
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
- Never remote staging MySQL, staging Redis, cloud DB proxies for this flow, or production  

Templates & mapping: [reference.md](reference.md).
