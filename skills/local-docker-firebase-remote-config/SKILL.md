---
name: local-docker-firebase-remote-config
description: >-
  Start, seed, verify, and inspect local Firebase Remote Config for feature-flag
  tests (Docker mock + optional Redis FF cache). Use when the user wants local
  Remote Config / feature flags, ticket-scoped flag seeds, or preflight before
  APIs gated by Remote Config. Prefer Redis cache seed for Go clients that call
  the Google RC API; Docker mock on :9299 for clients that honor
  FIREBASE_REMOTE_CONFIG_URL_BASE. Never use real staging/prod Remote Config
  unless the user explicitly overrides.
---

# Local Docker Firebase Remote Config

## Token rules (do these first)

1. **No official Firebase Remote Config emulator** — Firebase Emulator Suite does not include RC. Use this skill’s Docker mock and/or Redis cache seed.
2. **Do not** print Firebase private keys or service-account JSON in chat or reports.
3. Prefer **Redis FF cache seed** when the app caches Remote Config in Redis (common Go pattern) — cache hit avoids live Firebase.
4. Use Docker mock (`:9299`) when clients honor `FIREBASE_REMOTE_CONFIG_URL_BASE` (e.g. firebase-admin / REST).
5. Seed only flags the case needs; discover keys from project code — do not invent universal flag names.
6. Long recipes → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Local feature-flag / Remote Config seeding | Flows with no feature flags |
| Preflight before flag-gated APIs | Real Firebase RC unless user overrides |
| REST mock for Admin SDK / tools | MySQL/Redis/Dynamo/Firestore/Pub/Sub → sibling skills |

## Defaults (rename per project)

| Item | Default |
|------|---------|
| RC mock container | `app-firebase-rc-local` |
| RC mock port | `9299` |
| RC base URL | `http://127.0.0.1:9299` |
| Local project ID | `local-test` |
| Redis companion | `app-redis-local` via `local-docker-redis` |
| Cache key env | `FEATURE_FLAG_CACHE_KEY` (or project equivalent) |

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. Start Redis if the app caches FF templates there
- [ ] 3. Seed cache key and/or start RC mock with initialTemplate
- [ ] 4. Set FIREBASE_REMOTE_CONFIG_URL_BASE when using the mock (Admin SDK / REST)
- [ ] 5. Raise cache TTL for long local sessions if applicable
- [ ] 6. Verify GET cache / curl remoteConfig
- [ ] 7. Note remote_config_seeded in /tmp/<case>-seed.json
```

## Path A — Redis seed (Go / cache-first apps)

```bash
CACHE_KEY="${FEATURE_FLAG_CACHE_KEY:-FEATURE_FLAG_CACHE_KEY}"

docker exec app-redis-local redis-cli SET "$CACHE_KEY" '{
  "parameters": {
    "<flag_key>": {"defaultValue": {"value": "true"}}
  }
}'

docker exec app-redis-local redis-cli GET "$CACHE_KEY"
```

`value` is always a **string** (`"true"` / `"false"` / `"3"` / stringified JSON).

## Path B — Docker RC mock (REST / Admin SDK)

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

export FIREBASE_REMOTE_CONFIG_URL_BASE=http://127.0.0.1:9299
curl -s "http://127.0.0.1:9299/v1/projects/local-test/remoteConfig" | head -c 400
```

```bash
curl -s -X PUT "http://127.0.0.1:9299/v1/projects/local-test/remoteConfig" \
  -H 'Content-Type: application/json' -H 'If-Match: *' \
  -d '{"parameters":{"example_flag":{"defaultValue":{"value":"false"}}}}'
```

## Env alignment

| Pattern | Local value |
|---------|-------------|
| `FEATURE_FLAG_CACHE_KEY` | Project cache key; seed that Redis key |
| `FEATURE_FLAG_CACHE_TIME` | Prefer larger TTL for long local runs |
| `FIREBASE_REMOTE_CONFIG_URL_BASE` | `http://127.0.0.1:9299` (Admin SDK / REST) |
| `FIREBASE_PROJECT_ID` | `local-test` (or project-specific local id) |

## Related skills

| Skill | Role |
|-------|------|
| `local-docker-redis` | FF template cache companion |
| `test-data-conventions` | Ticket tags in seed notes |
| `local-docker-mysql` / `local-docker-pubsub` | Common API companions |
| `python-local-api-test` | Local API + result report |

## Forbidden (unless user explicitly overrides)

- Hitting staging/prod Remote Config when local cache/mock is available
- Pasting Firebase private keys into chat or reports
- Hard-coding one company’s flag names as universal defaults in reports
- Claiming Google Go RC clients auto-route to `:9299` without Redis seed or `WithEndpoint`

Details: [reference.md](reference.md).
