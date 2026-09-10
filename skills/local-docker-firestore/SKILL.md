---
name: local-docker-firestore
description: >-
  Start, health-check, seed documents, verify, and inspect the local Firestore
  emulator (Docker or gcloud) for API and integration tests. Use when the user
  wants local Firestore, emulator seeding, ticket-scoped document checks, or
  preflight before local API tests that call Firestore. Always set
  FIRESTORE_EMULATOR_HOST. Never use real cloud Firestore unless the user
  explicitly overrides.
---

# Local Docker Firestore

## Token rules (do these first)

1. **Emulator only** — default `FIRESTORE_EMULATOR_HOST=127.0.0.1:8080` via `app-firestore-local` (or `gcloud emulators firestore`).
2. Client SDKs and services **must** see the emulator host env; empty host usually means **real Firestore**.
3. **Do not** print service-account private keys in chat or reports.
4. Seed collection/document paths from the **project’s real code** — do not invent unrelated hierarchies.
5. Long recipes → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Start / health / seed / read emulator | Cases that never touch Firestore |
| Ticket-scoped document verify | Real cloud Firestore unless user overrides |
| Preflight for services with emulator host set | MySQL/Redis/Dynamo → sibling skills |

## Defaults (rename per project)

| Item | Default |
|------|---------|
| Container | `app-firestore-local` |
| Port | `8080` |
| Emulator host env | `FIRESTORE_EMULATOR_HOST=127.0.0.1:8080` |
| Project id | whatever the app already uses locally (placeholder `demo-project` is fine for pure emulator) |

## Checklist

```
- [ ] 1. Docker daemon up (or gcloud emulator installed)
- [ ] 2. Start/verify emulator (HTTP reachability on 8080)
- [ ] 3. Export FIRESTORE_EMULATOR_HOST=127.0.0.1:8080 for the API process
- [ ] 4. Seed docs only when the code path reads them
- [ ] 5. Verify by reading the same collection/doc
- [ ] 6. Note stores_seeded in /tmp/<case>-seed.json
```

## Start + health (Docker)

```bash
docker start app-firestore-local 2>/dev/null || \
docker run -d --name app-firestore-local -p 8080:8080 \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators \
  gcloud beta emulators firestore start --host-port=0.0.0.0:8080 --project=demo-project

curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080
```

## Alternative: gcloud on host

```bash
gcloud emulators firestore start --host-port=127.0.0.1:8080
export FIRESTORE_EMULATOR_HOST=127.0.0.1:8080
```

## Seed + verify (summary)

Use the project’s language SDK with emulator host set. Example (Python):

```bash
export FIRESTORE_EMULATOR_HOST=127.0.0.1:8080
# pip install google-cloud-firestore
python - <<'PY'
import os
from google.cloud import firestore
os.environ['FIRESTORE_EMULATOR_HOST'] = '127.0.0.1:8080'
db = firestore.Client(project='demo-project')
ref = db.collection('seed').document('<case>')
ref.set({'case': '<case>', 'ok': True})
print(ref.get().to_dict())
PY
```

Many cases are MySQL/Redis-only — an unused emulator is fine; still report health when the user asks for “all Docker”.

## Env alignment

| Pattern | Local value |
|---------|-------------|
| `FIRESTORE_EMULATOR_HOST` | `127.0.0.1:8080` |
| Empty | Treat as real Firestore — forbidden for local tests |

## Related skills

| Skill | Role |
|-------|------|
| `local-docker-mysql` / `local-docker-redis` / `local-docker-dynamodb` | Other local stores |
| `playwright-local-api-test` | Local API + result report |

## Forbidden (unless user explicitly overrides)

- Writing to production/staging Firestore projects
- Running clients without `FIRESTORE_EMULATOR_HOST` during “local” tests
- Pasting private key PEM material into skills, chat, or reports
