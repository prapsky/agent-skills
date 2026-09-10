---
name: playwright-local-api-test
description: >-
  Run local Playwright API tests against Docker MySQL + Docker Redis, with seed
  data from mysql-insert, write HTML reports, and always deliver the standard
  local-test result report (chat + result.html; auto PR comment when a PR is in
  context). Use when testing locally with Playwright, verifying an API with
  seeded MySQL data, or reporting results. Never use remote staging MySQL or
  Redis.
---

# Playwright — local API test

## Token rules (do these first)

1. **Reuse** the repo’s existing `playwright/local-api/` (or equivalent) folder — do not scaffold a new project.
2. **Reuse** `/tmp/<case>-seed.json` + `/tmp/<case>-body.json` from `mysql-insert`.
3. **Do not** curl a mutating endpoint and then run Playwright on the **same** consumed seed without reseed (second call may hang/time out).
4. **Local Docker MySQL + Redis only** — `app-mysql-local` + `app-redis-local`. Never cloud DB proxies or remote staging DB/Redis. See `mysql-insert` + [reference.md](reference.md).
5. Keep the chat lead-in short (pass/fail + URLs), then **always** append the [Result report format](#result-report-format-mandatory). Details → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Local API test + HTML report + **mandatory result report** | SQL-only → `mysql-insert` |
| Auto PR comment with local proof when a PR is in context | UI E2E unless asked |
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
- [ ] 7. Write playwright-report/result.html using the Result report format
- [ ] 8. Always deliver the Result report in chat (same sections)
- [ ] 9. If a PR is in context: post the same Result report as a PR comment (unless user said not to)
```

## Result report format (mandatory)

**Whenever local testing finishes** (Playwright and/or local API probe after `mysql-insert` seed), always produce this report:

1. In chat (after the short pass/fail line)
2. In `playwright-report/result.html`
3. As a PR comment when a PR URL/number/branch PR is in context (auto; skip only if the user says not to comment)

Use these **exact section headings**. Request body and response must be **multi-line JSON**, never one line. No secrets.

```markdown
### The issue
<what was wrong for the user / product>
**Issue log:**
\`\`\`json
{ ... evidence of the problem / seed before fix ... }
\`\`\`

### What is the root cause
<why it happened>

### What you actually did in this PR
<what this change does to fix it>

### The local endpoint URL
`<full local URL>`

### The request body
\`\`\`json
{
  ...
}
\`\`\`

### The response
\`\`\`json
{
  ...
}
\`\`\`

### The explanation of the result
<one simple sentence>
```

Notes:

- GET endpoints: still show a multi-line JSON object (`method`, `path`, `headers` with secrets redacted, `body: null`) — do not omit the section.
- Failed runs: same sections; explanation states what failed in one sentence; mark PASSED/FAILED in `result.html`.
- Template example: [reference.md](reference.md#result-report-template).

## Security

- No secrets in HTML, attachments, chat reports, or PR comments  
- **Local Docker MySQL + Redis only** unless user explicitly asks otherwise  
- Never remote staging MySQL, staging Redis, cloud DB proxies for this flow, or production  

Templates & mapping: [reference.md](reference.md).
