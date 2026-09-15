---
name: local-docker-redis
description: >-
  Use this skill whenever you need to start, seed, verify, or inspect local
  Docker Redis for cache-backed API tests or integration tests. Triggers on:
  "seed Redis locally", "set Redis key", "local Docker cache", "PONG check",
  "ticket-scoped Redis scan", "preflight Redis before API test", or any request
  to prepare cache state before running a local HTTP test. Never use remote
  staging or production Redis — always default to app-redis-local on
  127.0.0.1:6379 unless the user explicitly overrides.
---

# Local Docker Redis

Start, seed, verify, and inspect the local Docker Redis container for
cache-backed API and integration tests. Keeping Redis local prevents accidental
cache poisoning in shared staging environments.

## When to use

| Use this skill | Skip and use instead |
|---------------|----------------------|
| Start / PING / inspect local Redis | Cases that never touch cache — no action needed |
| Seed a known key before a local API test | Remote Redis — only if user overrides |
| Ticket-scoped key scan before/after API | MySQL/Dynamo/Firestore → sibling skills |
| Flush or delete seed keys | `FLUSHALL` on non-local — Forbidden |

## Do these first (token rules)

1. Default to **local Docker only** — `app-redis-local` on `127.0.0.1:6379`. Never remote staging Redis.
2. Tag seeded keys as `seed:<ticket-or-case>:…` so you can clean them up safely without touching unrelated keys.
3. Prefer `--scan` over `KEYS *` on large DBs — `KEYS *` blocks the server.
4. Phones and string IDs in keys or payloads → follow [`test-data-conventions`](../test-data-conventions/SKILL.md).
5. Do not print Redis passwords in chat or reports (local default has none anyway).

## Defaults

| Item | Default |
|------|---------|
| Container | `app-redis-local` |
| Port | `6379` |
| Password | none |
| Image | `redis:7-alpine` |

## Workflow

1. **Start the container** (idempotent):

   ```bash
   docker start app-redis-local 2>/dev/null || \
   docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine
   ```

2. **Verify health:**

   ```bash
   docker exec app-redis-local redis-cli ping
   # expect: PONG
   ```

3. **Align the app's local env** — set `REDIS_HOST=127.0.0.1`, `REDIS_PORT=6379`, `REDIS_PASSWORD=` (empty) so the service under test talks to this container.

4. **Seed keys** the case needs (many API cases write cache themselves — an empty scan is valid):

   ```bash
   docker exec app-redis-local redis-cli SET 'seed:<case>:example' '{"ok":true}'
   docker exec app-redis-local redis-cli GET 'seed:<case>:example'
   ```

5. **Verify with a scan** scoped to the ticket:

   ```bash
   docker exec app-redis-local redis-cli --scan --pattern 'seed:<case>*'
   docker exec app-redis-local redis-cli --scan --pattern '*<phone-or-id>*'
   ```

   Report the scan result even if empty — it is evidence the store is clean before the test.

6. **Record in `/tmp/<case>-seed.json`** when Redis is part of a multi-store case:

   ```json
   {
     "stores_seeded": ["redis"],
     "redis_key_prefix": "seed:<case>:"
   }
   ```

7. **Cleanup** — only when the user asks:

   ```bash
   docker exec app-redis-local redis-cli --scan --pattern 'seed:<case>*' | \
     while read -r k; do [ -n "$k" ] && docker exec app-redis-local redis-cli DEL "$k"; done

   # Nuclear (local only, user must explicitly ask):
   docker exec app-redis-local redis-cli FLUSHDB
   ```

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. app-redis-local healthy (PONG) on 127.0.0.1:6379
- [ ] 3. REDIS_HOST/PORT/(PASSWORD) aligned in local env
- [ ] 4. Seed keys added (or confirmed not needed)
- [ ] 5. Ticket-scoped scan run and reported
- [ ] 6. FLUSHDB / DEL deferred unless user asks
```

## Examples

**Typed GET helpers for debugging:**

```bash
docker exec app-redis-local redis-cli TYPE '<key>'
docker exec app-redis-local redis-cli GET '<key>'           # string
docker exec app-redis-local redis-cli HGETALL '<key>'       # hash
docker exec app-redis-local redis-cli LRANGE '<key>' 0 -1   # list
docker exec app-redis-local redis-cli ZRANGE '<key>' 0 -1 WITHSCORES  # zset
```

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Phone `+6285YYMMDDxxx` + UUID string IDs |
| `local-docker-mysql` | Relational seed companion |
| `local-docker-dynamodb` / `local-docker-firestore` | Other local stores |
| `local-docker-firebase-remote-config` | Feature-flag cache seed (also uses Redis) |
| `python-local-api-test` | Local HTTP API probe + result report |

## Forbidden

- Staging or production Redis hosts or managed Redis URLs — local accidents can corrupt shared cache
- `FLUSHALL` / `FLUSHDB` on non-local instances — this wipes every key for every user
- Logging secrets from Redis values that contain tokens or private payloads

## References

Read `references/reference.md` for: optional `--requirepass` setup, typed-get helpers, env alignment table, and seed JSON conventions.
