---
name: local-docker-redis
description: >-
  Start, health-check, seed keys, verify, and inspect local Docker Redis for
  cache-backed API and integration tests. Use when the user wants local Redis,
  Docker Redis seeding, ticket-scoped key scans, or preflight before Playwright.
  Never use remote staging or production Redis unless the user explicitly
  overrides.
---

# Local Docker Redis

## Token rules (do these first)

1. **Local Docker only** — default container `app-redis-local` on `127.0.0.1:6379`. Never remote staging/prod Redis.
2. **Do not** print Redis passwords in chat or reports (default local has none).
3. Prefer `--scan` over `KEYS *` on large DBs.
4. Tag seeded keys with `seed:<ticket-or-case>:…` so cleanup and inspect stay safe.
5. **Phones / string IDs** in keys or payloads → [`test-data-conventions`](../test-data-conventions/SKILL.md).
6. Long recipes → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Start / PING / inspect local Redis | Cases that never touch cache |
| Seed a known key for a local test | Remote Redis unless user overrides |
| Ticket-scoped key scan before/after API | MySQL/Dynamo/Firestore → sibling skills |

## Defaults (rename per project)

| Item | Default |
|------|---------|
| Container | `app-redis-local` |
| Port | `6379` |
| Password | none |

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. Start/verify app-redis-local (PONG)
- [ ] 3. Align REDIS_HOST/PORT/(PASSWORD) in local env to Docker
- [ ] 4. Seed keys only when the case needs them
- [ ] 5. Verify with --scan --pattern '*<case>*'
- [ ] 6. FLUSHDB / DEL only if user asks (local container only)
```

## Start + health

```bash
docker start app-redis-local 2>/dev/null || \
docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine

docker exec app-redis-local redis-cli ping
# expect: PONG
```

## Seed + verify

```bash
docker exec app-redis-local redis-cli SET 'seed:<case>:example' '{"ok":true}'
docker exec app-redis-local redis-cli GET 'seed:<case>:example'
docker exec app-redis-local redis-cli --scan --pattern 'seed:<case>*'
docker exec app-redis-local redis-cli --scan --pattern '*<phone-or-id>*'
```

Many API cases write cache themselves — an empty scan is valid; still run the check and report it.

## Cleanup

```bash
# Prefer deleting seed-prefixed keys
docker exec app-redis-local redis-cli --scan --pattern 'seed:<case>*' | \
  while read -r k; do [ -n "$k" ] && docker exec app-redis-local redis-cli DEL "$k"; done

# Nuclear (local only, user must ask)
docker exec app-redis-local redis-cli FLUSHDB
```

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Phone `+6285YYMMDDxxx` + UUID string IDs |
| `local-docker-mysql` | Relational seed companion |
| `python-local-api-test` | Local API + result report |
| `local-docker-dynamodb` / `local-docker-firestore` | Other local stores |

## Forbidden (unless user explicitly overrides)

- Staging / production Redis hosts or managed Redis URLs
- `FLUSHALL` / `FLUSHDB` on non-local instances
- Logging secrets from Redis values that contain tokens
