# Python local API test — reference

## Why Python (not Playwright) for APIs

| Concern | Prefer |
|---------|--------|
| HTTP request/response, status, JSON asserts | **Python** (`python-local-api-test`) |
| Browser UI, clicks, screenshots | Playwright |

## Env + token (generic)

Parse `.env.testing` / `.env.local` with Python. Mint JWTs with the project’s secret; write token length only to chat, never the secret or full token in reports.

## Pairing with Docker skills

1. `local-docker-mysql` (+ redis/dynamo/firestore as needed)  
2. Seed `/tmp/<case>-seed.json`  
3. Start API on Docker stack  
4. Python probe → result JSON + HTML + chat report  
5. Optional PR/issue comment (redacted)

## Result HTML

Mirror the Result report sections from `SKILL.md`. Mark PASSED/FAILED.

## Cleanup

Only if user asks: delete seed rows / keys tagged `seed:<case>%`.
