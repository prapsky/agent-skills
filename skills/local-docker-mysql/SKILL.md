---
name: local-docker-mysql
description: >-
  Use this skill whenever you need to start, seed, verify, or inspect local
  Docker MySQL for API tests, integration tests, or bug replication. Triggers on:
  "seed local MySQL", "insert test data", "local Docker database", "DBeaver SQL",
  "preflight before local API", "ticket-scoped SELECT", "mysql-insert", or any
  request to set up data before running a local HTTP test. If Python API testing
  follows, hand off to python-local-api-test. Never touch staging or production
  MySQL unless the user explicitly overrides — always default to app-mysql-local
  on 127.0.0.1:3306.
---

# Local Docker MySQL

Start, seed, verify, and inspect the local Docker MySQL container for API and
integration tests. This skill keeps all data local so you never accidentally
mutate shared staging data.

## When to use

| Use this skill | Skip and use instead |
|---------------|----------------------|
| Start / ping / inspect local MySQL | Unit tests with mocks — no Docker needed |
| Seed / INSERT / run DBeaver SQL locally | Staging MySQL — only if user overrides |
| Prep data before a Python local API run | Redis/Dynamo/Firestore → sibling skills |
| Ticket-scoped `SELECT` to verify seed tags | PR creation unless it adds SQL/Docker docs |

## Do these first (token rules)

1. Check `/tmp/<case>-seed.json` — reuse it when still valid instead of re-discovering the whole schema.
2. Parse local env files with **Python**, not `source` in zsh (zsh `source` does not reliably export to child processes).
3. Discover FK ids with `SELECT … WHERE <unique_col> = '…'` — never invent UUID foreign keys.
4. New phones and string PKs must follow [`test-data-conventions`](../test-data-conventions/SKILL.md) (`+6285YYMMDDxxx`, valid UUID PKs).
5. Do not print DB passwords in chat, HTML reports, or PR comments.
6. Default to **local Docker only** — `app-mysql-local` on `127.0.0.1:3306`. Never point at cloud SQL proxies or remote staging unless overridden.

## Defaults

| Item | Default |
|------|---------|
| Container | `app-mysql-local` |
| Port | `3306` |
| Database | `app_local` |
| User / password | `app` / `app_local` |
| Root password | `app_root_local` |
| Image | `mysql:8.0` |
| Auth plugin | `mysql_native_password` |

Rename container and credentials per project — keep Docker `MYSQL_*` and `.env.testing` / `.env.local` in sync.

## Workflow

1. **Start the container** (idempotent — safe to run even when already running):

   ```bash
   docker start app-mysql-local 2>/dev/null || \
   docker run -d --name app-mysql-local \
     -e MYSQL_DATABASE=app_local \
     -e MYSQL_USER=app \
     -e MYSQL_PASSWORD=app_local \
     -e MYSQL_ROOT_PASSWORD=app_root_local \
     -p 3306:3306 \
     mysql:8.0 \
     --default-authentication-plugin=mysql_native_password
   ```

2. **Verify health:**

   ```bash
   docker exec app-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -papp_root_local --silent
   ```

3. **Align the app's local env** (`.env.testing` / `.env.local`) so `MYSQL_HOST=127.0.0.1`, `MYSQL_PORT=3306`, and credentials match the container — do not point the API at remote staging for seed/test runs.

4. **Discover required FK ids** before seeding. Do not invent UUIDs for foreign keys:

   ```sql
   SELECT id FROM <fk_table> WHERE <unique_name_col> = '<known-fixture-name>' LIMIT 1;
   ```

5. **Seed with `INSERT … SELECT`** to keep FK lookups correct. Tag the row so cleanup is safe:

   ```sql
   -- New string PKs: valid UUIDs  |  Phones: +6285YYMMDDxxx (see test-data-conventions)
   INSERT INTO <table> (<cols…>)
   SELECT
     '<new-uuid>', '<+6285YYMMDDxxx>', parent.id, 'seed:<ticket>'
   FROM <fk_table> parent
   WHERE parent.<unique_col> = '<known-fixture-name>'
   LIMIT 1;
   ```

6. **Verify** the inserted row:

   ```bash
   docker exec app-mysql-local mysql -uapp -papp_local app_local -e \
     "SELECT * FROM <table> WHERE remarks LIKE 'seed:<TICKET>%' LIMIT 5;"
   ```

7. **Write `/tmp/<case>-seed.json`** with key fields (no secrets). This JSON is the handoff to downstream skills.

8. **Pair with Redis** when the API needs cache:

   ```bash
   docker start app-redis-local 2>/dev/null || \
   docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine
   docker exec app-redis-local redis-cli ping   # expect: PONG
   ```

9. **Continue to Python API test** — follow `python-local-api-test` and deliver its mandatory result report.

10. **Cleanup** — only when the user asks:

    ```bash
    docker exec app-mysql-local mysql -uapp -papp_local app_local -e \
      "DELETE FROM <table> WHERE remarks LIKE 'seed:<TICKET>%';"
    ```

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. app-mysql-local healthy on 127.0.0.1:3306
- [ ] 3. app-redis-local healthy (PONG) — when the API needs cache
- [ ] 4. Local env aligned to 127.0.0.1:3306 (MYSQL_*)
- [ ] 5. FK ids discovered by real unique keys (not invented)
- [ ] 6. New string PKs are valid UUIDs; phones are +6285YYMMDDxxx
- [ ] 7. INSERT + verify SELECT succeeded
- [ ] 8. /tmp/<case>-seed.json written (no secrets)
- [ ] 9. Cleanup DELETE deferred unless user asks
```

## Examples

**Example seed JSON handoff (`/tmp/acq-2937-seed.json`):**

```json
{
  "case": "acq-2937",
  "stores_seeded": ["mysql"],
  "mysql": {
    "table": "leads",
    "id": "a3f1c2e4-9b8d-4e2a-91f0-7c6d5b4a3210",
    "phone": "+6285260915001",
    "tag": "seed:acq-2937%"
  }
}
```

**Example pymysql fallback** (when Homebrew `mysql` fails auth plugin):

```bash
python3 -m venv /tmp/seed-venv && /tmp/seed-venv/bin/pip install -q pymysql
# then run your INSERT script inside /tmp/seed-venv/bin/python
# host=127.0.0.1 port=3306 user=app password=app_local database=app_local
```

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Phone `+6285YYMMDDxxx` + UUID string IDs |
| `local-docker-redis` | Cache companion for API tests |
| `local-docker-dynamodb` | Local Dynamo when the case needs it |
| `local-docker-firestore` | Firestore emulator when the case needs it |
| `python-local-api-test` | Local HTTP API probe + result report |

## Forbidden

- Cloud SQL proxies or managed-DB connections pointing at remote staging — use local Docker instead (accidents in staging are costly and hard to reverse)
- Seeding against remote staging or production MySQL hosts
- Printing DB passwords in chat, HTML reports, or PR/issue comments
- Committing real credentials into skill files or reports

## References

Read `references/reference.md` for: DBeaver tips, pymysql setup, dump/restore commands, env alignment table, and typed SQL examples.
