---
name: local-docker-firebase-remote-config
description: >-
  Use this skill whenever you need to seed, verify, or mock local Firebase
  Remote Config for feature-flag tests. Triggers on: "seed feature flag locally",
  "local Remote Config", "FEATURE_FLAG_CACHE_KEY", "firebase-mocker", "flag-gated
  API local test", "enable feature flag for local test", "FIREBASE_REMOTE_CONFIG_URL_BASE",
  or any request to control a Remote Config / feature-flag value before running
  a local HTTP test. Two paths: Redis cache seed (most Go apps) or Docker
  firebase-mocker on :9299 (Admin SDK / REST clients). Never hit staging or
  production Remote Config unless the user explicitly overrides.
---

# Local Docker Firebase Remote Config

Seed and inspect local Firebase Remote Config for feature-flag-gated API tests.
There is no official Firebase RC emulator — this skill covers the two practical
alternatives so you can control flag values without touching real Firebase.

## When to use

| Use this skill | Skip and use instead |
|---------------|----------------------|
| Seed a flag before a flag-gated local API test | Flows with no feature flags — no action needed |
| Preflight before APIs that read RC via Redis cache | Real Firebase RC — only if user overrides |
| REST mock for Admin SDK / firebase-admin tools | MySQL/Redis/Dynamo/Firestore/Pub/Sub → sibling skills |

## Why two paths?

| Path | When to use | Why it works |
|------|-------------|--------------|
| **Path A — Redis cache seed** | App caches the RC template JSON in Redis (common Go pattern) | Cache hit → no live Firebase call at all |
| **Path B — Docker `firebase-mocker` `:9299`** | Client honours `FIREBASE_REMOTE_CONFIG_URL_BASE` (Admin SDK / REST) | Community HTTP mock of the RC REST API |

Official Firebase Emulator Suite does **not** include Remote Config.

## Do these first (token rules)

1. Discover which path the project uses — look for `FEATURE_FLAG_CACHE_KEY` (Redis) or `FIREBASE_REMOTE_CONFIG_URL_BASE` (HTTP mock) in the project env files.
2. Seed only the flags the case needs — discover key names from project code, do not invent universal flag names.
3. `value` in the RC JSON is always a **string** (`"true"` / `"false"` / `"3"` / stringified JSON) — even boolean flags are stored as the string `"true"`.
4. Do not print Firebase private keys or service-account JSON in chat or reports.
5. Follow [`local-docker-redis`](../local-docker-redis/SKILL.md) for Redis startup before Path A.

## Defaults

| Item | Default |
|------|---------|
| RC mock container | `app-firebase-rc-local` |
| RC mock port | `9299` |
| RC base URL env | `FIREBASE_REMOTE_CONFIG_URL_BASE=http://127.0.0.1:9299` |
| Local project ID | `local-test` |
| Redis companion | `app-redis-local` via `local-docker-redis` |
| Cache key env | `FEATURE_FLAG_CACHE_KEY` (discover project-specific name) |

## Workflow — Path A: Redis cache seed

Use this path when the app reads a Redis key containing the RC template JSON (avoids a live Firebase network call entirely).

1. **Start Redis** (follow `local-docker-redis` for full setup):

   ```bash
   docker start app-redis-local 2>/dev/null || \
   docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine
   docker exec app-redis-local redis-cli ping   # expect: PONG
   ```

2. **Discover the cache key name** from the project env (`FEATURE_FLAG_CACHE_KEY` or equivalent).

3. **Seed the flag template** (value is always a string):

   ```bash
   CACHE_KEY="${FEATURE_FLAG_CACHE_KEY:-FEATURE_FLAG_CACHE_KEY}"

   docker exec app-redis-local redis-cli SET "$CACHE_KEY" '{
     "parameters": {
       "<flag_key>": {"defaultValue": {"value": "true"}}
     }
   }'
   ```

4. **Verify the seed:**

   ```bash
   docker exec app-redis-local redis-cli GET "$CACHE_KEY"
   ```

5. (Optional) Raise `FEATURE_FLAG_CACHE_TIME` in local env to avoid TTL expiry during a long test session.

## Workflow — Path B: Docker RC mock

Use this path when the client honours `FIREBASE_REMOTE_CONFIG_URL_BASE` and calls the RC REST API directly.

1. **Start the mock container** (uses `firebase-mocker@2`):

   ```bash
   docker start app-firebase-rc-local 2>/dev/null || \
   docker run -d --name app-firebase-rc-local -p 9299:9299 -w /app node:22-bookworm-slim \
     bash -c 'npm init -y && npm i firebase-mocker@2 && node -e "
   const { firebaseMocker } = require(\"firebase-mocker\");
   firebaseMocker.startRemoteConfigServer({
     port: 9299,
     host: \"0.0.0.0\",
     projectId: \"local-test\",
     initialTemplate: {
       parameters: {
         example_flag: { defaultValue: { value: \"true\" }, valueType: \"BOOLEAN\" }
       }
     }
   }).then(() => console.log(\"RC mock on :9299\"));
   "'
   ```

2. **Export the base URL** for the service under test:

   ```bash
   export FIREBASE_REMOTE_CONFIG_URL_BASE=http://127.0.0.1:9299
   ```

3. **Verify the mock is serving:**

   ```bash
   curl -s "http://127.0.0.1:9299/v1/projects/local-test/remoteConfig" | head -c 400
   ```

4. **Update a flag at runtime** (no container restart needed):

   ```bash
   curl -s -X PUT "http://127.0.0.1:9299/v1/projects/local-test/remoteConfig" \
     -H 'Content-Type: application/json' -H 'If-Match: *' \
     -d '{"parameters":{"<flag_key>":{"defaultValue":{"value":"false"}}}}'
   ```

## Env alignment

| Pattern | Local value |
|---------|-------------|
| `FEATURE_FLAG_CACHE_KEY` | Project cache key name (discover from code/env) |
| `FEATURE_FLAG_CACHE_TIME` | Prefer a large TTL (e.g. `3600`) for long local runs |
| `FIREBASE_REMOTE_CONFIG_URL_BASE` | `http://127.0.0.1:9299` (for Path B) |
| `FIREBASE_PROJECT_ID` | `local-test` |

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. Redis started (Path A) or RC mock started (Path B)
- [ ] 3. Flag seed written to Redis / mock with correct string values
- [ ] 4. Env aligned (FEATURE_FLAG_CACHE_KEY / FIREBASE_REMOTE_CONFIG_URL_BASE)
- [ ] 5. Verify GET cache / curl remoteConfig confirms seed
- [ ] 6. /tmp/<case>-seed.json updated with remote_config_seeded
```

## Related skills

| Skill | Role |
|-------|------|
| `local-docker-redis` | FF template cache companion (Path A) |
| `test-data-conventions` | Ticket tags in seed notes |
| `local-docker-mysql` / `local-docker-pubsub` | Common API companions |
| `python-local-api-test` | Local HTTP API probe + result report |

## Forbidden

- Hitting staging or production Remote Config when local cache or mock is available — flag changes in production affect real users
- Pasting Firebase private keys or service-account JSON into chat or reports
- Hard-coding one project's flag names as universal defaults in this skill
- Claiming that Go RC clients auto-route to `:9299` — they do not without Redis seed or a custom `WithEndpoint` option

## References

Read `references/reference.md` for: Redis cache JSON schema, `firebase-mocker` version notes, cleanup commands, and known limitations of the mock vs production RC.
