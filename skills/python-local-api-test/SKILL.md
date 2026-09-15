---
name: python-local-api-test
description: >-
  Use this skill whenever you need to run a local HTTP API test, replicate a bug
  via HTTP, or deliver a result report after a local API run. Triggers on: "test
  local API", "local HTTP probe", "replicate bug locally", "run API test against
  Docker", "result report for local test", "call local endpoint", or any request
  to send HTTP requests to a locally running service. Always use Python (not
  Playwright) for HTTP API checks — Playwright is for browser UI only. Always
  deliver the mandatory result report. Never use remote staging databases unless
  the user explicitly overrides.
---

# Python — local API test

Run local HTTP API tests with Python against Docker-backed services and deliver
a standardised result report. Using Python (not Playwright) for API checks keeps
the flow fast, headless, and free of browser dependencies.

## When to use

| Use this skill | Skip and use instead |
|---------------|----------------------|
| Local HTTP API probe / regression / bug replication | SQL-only seed with no HTTP call → `local-docker-mysql` |
| Mandatory result report after local API run | Browser UI flows, clicks, screenshots → `playwright-local-api-test` |
| Auto PR/issue comment with local API proof | Staging/prod hosts — only if user overrides |

## Do these first (token rules)

1. **Use Python, not Playwright**, for local HTTP API checks — Playwright is browser-only.
2. **Reuse `/tmp/<case>-seed.json`** (and optional `/tmp/<case>-body.json`) from `local-docker-mysql` / other store skills instead of re-seeding.
3. Do not call a mutating endpoint twice on the same consumed seed without reseeding first.
4. Parse local env files and mint JWTs with **Python** — never `source` in zsh (zsh source does not reliably export to child processes).
5. New phones and string IDs in request bodies → follow [`test-data-conventions`](../test-data-conventions/SKILL.md).
6. Never print JWT secrets, env passwords, or full tokens in chat or reports — log token length only.

## Defaults

| Item | Typical value |
|------|---------------|
| Client | Python 3 + stdlib `urllib` (or `httpx` / `requests` in venv) |
| Script | `/tmp/<case>-api-probe.py` or `scripts/local_api/<case>.py` |
| Result JSON | `/tmp/<case>-api-result.json` |
| Result HTML | `/tmp/<case>-result.html` |
| MySQL | `app-mysql-local` → `127.0.0.1:3306` |
| Redis | `app-redis-local` → `127.0.0.1:6379` |
| Env source | Project `.env.testing` / `.env.local` via Python |
| Auth | Mint local JWT from the project's token secret when routes need Bearer |

## Workflow

1. **Preflight — start local Docker stores:**

   ```bash
   docker start app-mysql-local app-redis-local 2>/dev/null || true
   docker exec app-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -papp_root_local --silent
   docker exec app-redis-local redis-cli ping
   ```

   Add Dynamo, Firestore, or Pub/Sub when the case needs them — follow the matching `local-docker-*` skills.

2. **Confirm the seed** is ready at `/tmp/<case>-seed.json`. If not, run `local-docker-mysql` (or the relevant store skill) first.

3. **Write the probe script** — use the stdlib pattern for single requests, table-driven for multiple cases:

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
   print("status", status)   # never print token
   ```

   **Table-driven pattern** (for multiple assertions): loop `{name, method, path, body, expect_status, expect}` and fail on mismatch.

4. **Write `/tmp/<case>-api-result.json`** from the probe output.

5. **Write `/tmp/<case>-result.html`** — include the result report sections, PASSED/FAILED banner, ISO timestamp, and a link to `./index.html`.

6. **Deliver the result report** in chat immediately after the probe (short pass/fail line, then the full report below).

7. **Post to PR/issue** — when a PR or issue is in context, auto-comment the same report (secrets redacted) unless the user says not to.

## Result report format (mandatory)

Produce this report every time a local API test finishes — in chat AND in `result.html`. Use these **exact section headings**. Multi-line JSON for request body and response. No secrets.

```markdown
### The issue
<what was wrong for the user / product>
**Issue log:**
\`\`\`json
{
  "id": "…",
  "problem": "…"
}
\`\`\`

### What is the root cause
<why it happened>

### What you actually did in this PR
<what the change does to fix it — or "local replication only" when no code fix>

### The local endpoint URL
`http://127.0.0.1:<port>/<path>`

### The request body
\`\`\`json
{
  "method": "GET",
  "path": "/v1/example",
  "headers": {
    "Authorization": "Bearer <redacted>",
    "Content-Type": "application/json"
  },
  "body": null
}
\`\`\`

### The response
\`\`\`json
{
  "data": {
    "example_field": "…"
  },
  "meta": {
    "status_code": 200
  }
}
\`\`\`

### The explanation of the result
<one simple sentence about pass or fail>
```

**Notes:**
- GET endpoints: still show the multi-line request JSON (`method`, `path`, `headers` redacted, `body: null`) — do not omit the section.
- Failed runs: same sections; explanation states what failed in one sentence; mark FAILED in `result.html`.

## Checklist

```
- [ ] 1. Docker stores up and healthy
- [ ] 2. Seed ready at /tmp/<case>-seed.json
- [ ] 3. API listening on local Docker stack
- [ ] 4. Python probe written (table-driven for multiple cases)
- [ ] 5. /tmp/<case>-api-result.json written
- [ ] 6. /tmp/<case>-result.html written (PASSED/FAILED banner)
- [ ] 7. Result report delivered in chat
- [ ] 8. PR/issue comment posted (redacted) unless user opted out
```

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Phone `+6285YYMMDDxxx` + UUID string IDs |
| `local-docker-mysql` / `redis` / `dynamodb` / `firestore` | Seed + preflight |
| `playwright-local-api-test` | UI browser flows only — redirect API work here |

## Forbidden

- Using Playwright for local HTTP API tests — it adds browser overhead and misses status codes
- Pointing at remote staging DB/Redis for a "local" API run
- Printing JWT secrets, env passwords, or full tokens in any output

## References

Read `references/reference.md` for: full result report template, env/JWT parsing helpers, `httpx` / `requests` venv setup, and PR comment automation notes.
