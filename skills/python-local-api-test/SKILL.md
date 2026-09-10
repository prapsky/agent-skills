---
name: python-local-api-test
description: >-
  Run local HTTP API tests with Python against Docker-backed services. Use
  whenever the user wants to test an API locally, replicate a bug via HTTP, or
  report local API results. Prefer Python over Playwright for API checks —
  Playwright is for UI. Always deliver the standard result report. Never use
  remote staging databases unless the user explicitly overrides.
---

# Python — local API test

## Token rules (do these first)

1. **Prefer Python for local API tests** — do **not** use Playwright for HTTP API checks (Playwright is for UI).
2. **Reuse** `/tmp/<case>-seed.json` (+ optional `/tmp/<case>-body.json`) from `local-docker-mysql` / other store skills.
3. **Do not** call a mutating endpoint twice on the same consumed seed without reseed.
4. **Local Docker only** — at minimum `app-mysql-local` + `app-redis-local`; add Dynamo/Firestore when needed. Never cloud DB proxies or remote staging stores.
5. Parse local env files / mint JWTs with **Python** — never `source` in zsh; never print secrets.
6. Keep the chat lead-in short (pass/fail + URL), then **always** append the [Result report format](#result-report-format-mandatory).

## When to use / skip

| Use | Skip |
|-----|------|
| Local HTTP API probe / regression / bug replication | SQL-only seed → `local-docker-mysql` |
| Mandatory result report after local API run | UI browser flows → Playwright |
| Auto PR / issue comment with local API proof | Staging/prod hosts unless user overrides |

## Defaults (rename per project)

| Item | Typical value |
|------|----------------|
| Client | Python 3 + stdlib `urllib` (or `httpx` / `requests` in a venv) |
| Script | `/tmp/<case>-api-probe.py` or repo `scripts/local_api/<case>.py` |
| Result JSON | `/tmp/<case>-api-result.json` |
| Result HTML | `/tmp/<case>-result.html` |
| MySQL | `app-mysql-local` → `127.0.0.1:3306` |
| Redis | `app-redis-local` → `127.0.0.1:6379` |
| Env | Project `.env.testing` / `.env.local` via Python |
| Auth | Mint local JWT from the project’s token secret when routes need Bearer |

### Preflight

```bash
docker start app-mysql-local app-redis-local 2>/dev/null || true
docker exec app-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -papp_root_local --silent
docker exec app-redis-local redis-cli ping
```

See `local-docker-mysql` / `local-docker-redis` (and Dynamo/Firestore siblings) for full recipes.

## Checklist

```
- [ ] 1. Docker stores up
- [ ] 2. Seed ready (/tmp/<case>-seed.json)
- [ ] 3. API listening on local Docker stack
- [ ] 4. Python probe (table-driven when multiple cases)
- [ ] 5. Write /tmp/<case>-api-result.json
- [ ] 6. Write result HTML using Result report format
- [ ] 7. Deliver Result report in chat
- [ ] 8. If PR/issue in context: post same report (secrets redacted) unless user said not to
```

## Probe pattern (stdlib)

```python
import json, urllib.request
from pathlib import Path

token = Path("/tmp/<case>-token.txt").read_text().strip()
url = "http://127.0.0.1:<port>/<path>"
req = urllib.request.Request(
    url,
    headers={"Authorization": f"Bearer {token}", "Content-Type": "application/json"},
    method="GET",
)
with urllib.request.urlopen(req, timeout=60) as resp:
    body = resp.read().decode()
    status = resp.status
Path("/tmp/<case>-api-result.json").write_text(
    json.dumps({"url": url, "status": status, "response": json.loads(body)}, indent=2)
)
print("status", status)  # never print token
```

**Table-driven:** loop `{name, method, path, body, expect_status, expect}` and fail on mismatch.

## Result report format (mandatory)

Exact section headings. Multi-line JSON only. No secrets.

```markdown
### The issue
<what was wrong>
**Issue log:**
\`\`\`json
{ ... }
\`\`\`

### What is the root cause
<why>

### What you actually did in this PR
<fix, or "local replication only">

### The local endpoint URL
`<full local URL>`

### The request body
\`\`\`json
{
  "method": "GET",
  "path": "/v1/example",
  "headers": { "Authorization": "Bearer <redacted>" },
  "body": null
}
\`\`\`

### The response
\`\`\`json
{ ... }
\`\`\`

### The explanation of the result
<one simple sentence>
```

## Related skills

| Skill | Role |
|-------|------|
| `local-docker-mysql` / `redis` / `dynamodb` / `firestore` | Seed + preflight |
| `playwright-local-api-test` (legacy name) | **UI only** — redirect API work here |

## Forbidden (unless user explicitly overrides)

- Playwright for local HTTP API tests
- Remote staging DB/Redis for “local” API runs
- Printing JWT secrets or env passwords

Details: [reference.md](reference.md).
