# Python local API test — reference

## Why Python (not Playwright) for APIs

| Concern | Prefer |
|---------|--------|
| HTTP request/response, status, JSON asserts | **Python** (`python-local-api-test`) |
| Browser UI, clicks, screenshots | Playwright |

## Env + token (generic)

Parse `.env.testing` / `.env.local` with Python. Mint JWTs with the project’s secret; write token length only to chat, never the secret or full token in reports.

## Result report template

Always use these headings after a local API test run (chat + `result.html` + PR/issue comment when in context):

```markdown
### The issue
…
**Issue log:**
```json
{
  "id": "…",
  "problem": "…"
}
```

### What is the root cause
…

### What you actually did in this PR
…

### The local endpoint URL
`http://127.0.0.1:<port>/<path>`

### The request body
```json
{
  "method": "GET",
  "path": "/v1/example",
  "headers": {
    "Authorization": "Bearer <redacted>",
    "Content-Type": "application/json"
  },
  "body": null
}
```

### The response
```json
{
  "data": {
    "example_field": "…"
  },
  "meta": {
    "status_code": 200
  }
}
```

### The explanation of the result
One simple sentence about pass or fail.
```

Write `/tmp/<case>-result.html` (or the project’s report path) with the same sections. Mark PASSED/FAILED.

## Pairing with Docker skills

1. `local-docker-mysql` (+ redis/dynamo/firestore as needed)  
2. Seed `/tmp/<case>-seed.json`  
3. Start API on Docker stack  
4. Python probe → result JSON + HTML + chat report (mandatory Result report format)  
5. Optional PR/issue comment (redacted)

## Cleanup

Only if user asks: delete seed rows / keys tagged `seed:<case>%`.
