---
name: playwright-local-api-test
description: >-
  Use this skill for local Playwright UI / browser automation tests only. Do NOT
  use for HTTP API checks — use python-local-api-test instead. Triggers on:
  "Playwright UI test", "browser E2E locally", "screenshot local page",
  "Playwright spec", or "browser automation". If the user says "test the API"
  or "local HTTP probe", redirect immediately to python-local-api-test — do not
  start Playwright for that. When UI flows create phones or string IDs, follow
  test-data-conventions.
---

# Playwright — UI test (not for HTTP API)

Run local Playwright tests for browser UI flows, page navigation, and
screenshots. Playwright is the right tool for user-facing interactions;
Python (`python-local-api-test`) is the right tool for HTTP API probes.

## When to use

| Use this skill | Skip — use instead |
|---------------|--------------------|
| UI E2E flows, page interactions, screenshots | Local HTTP API probe → `python-local-api-test` |
| Browser automation the user explicitly requests | SQL seed only → `local-docker-mysql` |
| Verifying rendered page state after an action | JSON response assertion → `python-local-api-test` |

## If the user asks for local API testing

Redirect immediately to **`python-local-api-test`**. Do not start Playwright for HTTP API probes — Playwright adds browser overhead and cannot directly assert HTTP status codes or JSON bodies the way Python can.

## Do these first

1. Confirm the Docker stores the UI depends on are healthy — follow `local-docker-mysql` and `local-docker-redis` for MySQL and Redis.
2. When the UI flow creates phones or string IDs, follow [`test-data-conventions`](../test-data-conventions/SKILL.md) (`+6285YYMMDDxxx`, valid UUIDs).
3. Seed data before the test run. Write `/tmp/<case>-seed.json` from the store skill so the test can read it.

## Workflow

1. **Start Docker stores** the UI needs:

   ```bash
   docker start app-mysql-local app-redis-local 2>/dev/null || true
   docker exec app-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -papp_root_local --silent
   docker exec app-redis-local redis-cli ping
   ```

2. **Seed** using the appropriate `local-docker-*` skill. Write `/tmp/<case>-seed.json`.

3. **Run the Playwright spec** against the local UI:

   ```bash
   cd "<repo>/playwright"
   PW_BASE_URL='http://127.0.0.1:<port>' \
   npx playwright test tests/<spec>.ts --reporter=list,html
   ```

4. **Deliver results** — include screenshots, pass/fail summary, and a note of which stores were seeded.

5. **Cleanup** — only if the user asks: delete seed rows or keys tagged with `seed:<case>`.

## Checklist

```
- [ ] 1. Docker stores healthy (MySQL, Redis, and others as needed)
- [ ] 2. Seed written to /tmp/<case>-seed.json
- [ ] 3. UI process running on local stack
- [ ] 4. Playwright spec run against local UI
- [ ] 5. Screenshots and pass/fail summary delivered
```

## Related skills

| Skill | Role |
|-------|------|
| `python-local-api-test` | HTTP API probes and result report — use this instead for API |
| `local-docker-mysql` | MySQL seed before UI test |
| `local-docker-redis` | Cache seed before UI test |
| `test-data-conventions` | Phone `+6285YYMMDDxxx` + UUID string IDs in UI flows |

## Forbidden

- Using Playwright for local HTTP API tests — Python handles that better and with less overhead
- Seeding or testing against remote staging databases during a "local" UI test run
- Running Playwright before the required Docker stores are confirmed healthy

## References

Read `references/reference.md` for: local Docker stack commands, env alignment table, and tips for pairing Playwright with seeded data.
