---
name: playwright-local-api-test
description: >-
  Run local Playwright API tests against a given endpoint using MySQL seed data
  (from the mysql-insert skill), then write an HTML result report. Use when the
  user asks to test locally with Playwright, verify an API with seeded MySQL
  data, or report Playwright results in HTML.
---

# Playwright — local API test

## When to use

- User wants to **test locally with Playwright**
- Inputs available (or can be gathered): **MySQL seed data** + **endpoint**
- User wants results in an **HTML file**

## When not to use

- Generating MySQL seed SQL only → use `mysql-insert`
- Browser UI E2E unless the user explicitly asks for UI flows
- During PR creation: **skip** unless the PR adds/changes Playwright tests

## Inputs

| Input | Source | Required |
|-------|--------|----------|
| **Seed data** | Output / verify rows from `mysql-insert` (ids, phone, names, FKs, etc.) | Yes |
| **Endpoint** | `METHOD` + full URL (or base URL + path) | Yes |
| Request body / headers | User or mapped from seed fields | If the endpoint needs them |
| Expected status / body checks | User (default: 2xx, no 5xx) | Optional |

If seed data is missing, run or ask for `mysql-insert` first, then continue.

### Seed data shape (from `mysql-insert`)

Accept whatever the insert verified, as a simple object. Example:

```json
{
  "table": "leads",
  "id": "d7b406af-a2d2-4237-88c3-1c1693e9153f",
  "phone": "6281268293775",
  "club": "FIT HUB BLOK M",
  "extra": {}
}
```

Map seed fields into the request body using names the user gives (e.g. `phone` → body field, `id` → path param). Do not invent fields.

## Outputs

Always write HTML under a local report folder (create if needed):

1. `playwright-report/index.html` — Playwright HTML reporter  
2. `playwright-report/result.html` — short human summary (required)

Return both paths to the user.

## Workflow

```
- [ ] 1. Confirm seed data + endpoint (+ expected status if given)
- [ ] 2. Confirm local service is reachable (quick curl/probe)
- [ ] 3. Create/update a small Playwright API test from inputs
- [ ] 4. Run Playwright with list + html reporters
- [ ] 5. Write result.html summary
- [ ] 6. Tell user pass/fail + report paths
```

### 1. Confirm inputs

Ask only if missing:

- Seed object (or “use the rows from mysql-insert just created”)
- `METHOD` + URL
- Body/headers mapping from seed
- Expected HTTP status (default `200`–`299`)

### 2. Local service

- Prefer user-provided base URL (e.g. `http://127.0.0.1:5007`)
- Prefer `.env.testing` when the user shared it for local runs
- Load env with a parser (Python/`dotenv`) — do **not** `source` `.env` in zsh when values contain `?` / special chars

### 3. Playwright test (API request)

Keep tests API-level with `request` fixture unless UI is requested.

Minimal pattern:

```ts
import { test, expect } from '@playwright/test';

const BASE = process.env.PW_BASE_URL!;
const METHOD = process.env.PW_METHOD || 'POST';
const PATH = process.env.PW_PATH!;
const BODY = JSON.parse(process.env.PW_BODY_JSON || '{}');
const EXPECT_STATUS = Number(process.env.PW_EXPECT_STATUS || '0'); // 0 = any 2xx

test('local API', async ({ request }) => {
  const res = await request.fetch(`${BASE}${PATH}`, {
    method: METHOD,
    data: ['GET', 'HEAD'].includes(METHOD) ? undefined : BODY,
    headers: { 'Content-Type': 'application/json' },
  });
  const status = res.status();
  const text = await res.text();
  await test.info().attach('seed', { body: process.env.PW_SEED_JSON || '{}', contentType: 'application/json' });
  await test.info().attach('response', { body: text, contentType: 'application/json' });
  if (EXPECT_STATUS > 0) expect(status, text).toBe(EXPECT_STATUS);
  else expect(status, text).toBeGreaterThanOrEqual(200), expect(status, text).toBeLessThan(300);
});
```

Project layout (default if none exists):

```text
playwright/local-api/
  package.json          # @playwright/test
  playwright.config.ts  # reporter: list + html → playwright-report
  tests/api.spec.ts
  playwright-report/
```

Place under the relevant service repo or workspace `playwright/` — reuse an existing Playwright folder when present.

### 4. Run

```bash
cd <playwright-project>
PW_BASE_URL='http://127.0.0.1:<port>' \
PW_METHOD='POST' \
PW_PATH='/v1/...' \
PW_BODY_JSON='{"...":"..."}' \
PW_SEED_JSON='{"id":"..."}' \
PW_EXPECT_STATUS='201' \
npx playwright test --reporter=list,html
```

### 5. `result.html` summary

Include:

- PASSED / FAILED
- Endpoint (`METHOD URL`)
- Seed data used (no secrets)
- Request body (redact tokens/passwords)
- Response status + body excerpt
- Timestamp
- Link to `./index.html`

Template: [reference.md](reference.md).

## Security

- Never put DB passwords, API keys, or `.env` secrets in HTML reports
- Do not commit `.env` / `.env.testing`
- Prefer staging/local endpoints only unless the user explicitly asks otherwise
