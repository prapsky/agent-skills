---
name: local-docker-firestore
description: >-
  Use this skill whenever you need to start, seed, verify, or inspect the local
  Firestore emulator for API and integration tests. Triggers on: "seed local
  Firestore", "Firestore emulator", "FIRESTORE_EMULATOR_HOST", "local Firestore
  document", "preflight Firestore before API", "ticket-scoped Firestore verify",
  or any request to prepare Firestore state before a local HTTP test. Always set
  FIRESTORE_EMULATOR_HOST — an empty host means real cloud Firestore (forbidden
  for local tests unless the user explicitly overrides).
---

# Local Docker Firestore

Start, seed, verify, and inspect the local Firestore emulator for API and
integration tests. Setting `FIRESTORE_EMULATOR_HOST` before starting the
client is what routes all SDK calls to the local emulator instead of real
cloud Firestore.

## When to use

| Use this skill | Skip and use instead |
|---------------|----------------------|
| Start / health / seed / read emulator docs | Cases that never touch Firestore |
| Ticket-scoped document verify | Real cloud Firestore — only if user overrides |
| Preflight for services that read `FIRESTORE_EMULATOR_HOST` | MySQL/Redis/Dynamo → sibling skills |

## Do these first (token rules)

1. Export `FIRESTORE_EMULATOR_HOST=127.0.0.1:8080` **before** client construction — the SDK only routes to the emulator when the env var is present at startup.
2. An empty or unset `FIRESTORE_EMULATOR_HOST` means the SDK talks to **real Firestore** — forbidden for local tests.
3. Do not print service-account private key PEM material in chat or reports.
4. Seed collection/document paths from the **project's real code** — do not invent unrelated hierarchies.
5. Phones and string IDs in documents → follow [`test-data-conventions`](../test-data-conventions/SKILL.md).

## Defaults

| Item | Default |
|------|---------|
| Container | `app-firestore-local` |
| Port | `8080` |
| Emulator host env | `FIRESTORE_EMULATOR_HOST=127.0.0.1:8080` |
| Project id | `demo-project` (any value works for pure emulator) |
| Image | `gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators` |

## Workflow

1. **Start the container** (idempotent):

   ```bash
   docker start app-firestore-local 2>/dev/null || \
   docker run -d --name app-firestore-local -p 8080:8080 \
     gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators \
     gcloud beta emulators firestore start --host-port=0.0.0.0:8080 --project=demo-project
   ```

   **Alternative — host `gcloud`** (when Docker is heavy):

   ```bash
   gcloud emulators firestore start --host-port=127.0.0.1:8080
   ```

2. **Verify health** (any HTTP response confirms the port is accepting):

   ```bash
   curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080
   ```

3. **Export the emulator host** for the API process and any seed script:

   ```bash
   export FIRESTORE_EMULATOR_HOST=127.0.0.1:8080
   ```

   Also set this in `.env.testing` / `.env.local` so child processes inherit it automatically.

4. **Seed documents** using the project's SDK (all official clients honour `FIRESTORE_EMULATOR_HOST`):

   ```bash
   export FIRESTORE_EMULATOR_HOST=127.0.0.1:8080
   python3 - <<'PY'
   import os
   from google.cloud import firestore
   db = firestore.Client(project='demo-project')
   ref = db.collection('<collection>').document('<case>')
   ref.set({'case': '<case>', 'tag': 'seed:<ticket>', 'ok': True})
   print(ref.get().to_dict())
   PY
   ```

5. **Verify** by reading the same path with the SDK or via REST:

   ```bash
   curl -s "http://127.0.0.1:8080/v1/projects/demo-project/databases/(default)/documents/<collection>/<doc>"
   ```

6. **Record in `/tmp/<case>-seed.json`:**

   ```json
   {
     "stores_seeded": ["firestore"],
     "firestore_emulator": "127.0.0.1:8080",
     "firestore_paths": ["<collection>/<doc>"]
   }
   ```

   Many cases are MySQL/Redis-only — an unused emulator is fine; still report health when the user asks for "all Docker".

## Checklist

```
- [ ] 1. Docker daemon up (or gcloud emulator installed)
- [ ] 2. app-firestore-local healthy on 127.0.0.1:8080
- [ ] 3. FIRESTORE_EMULATOR_HOST=127.0.0.1:8080 exported before client/API starts
- [ ] 4. Documents seeded (or confirmed not needed)
- [ ] 5. Verify by reading seeded collection/doc
- [ ] 6. /tmp/<case>-seed.json written
```

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Phone `+6285YYMMDDxxx` + UUID string IDs |
| `local-docker-mysql` / `local-docker-redis` / `local-docker-dynamodb` | Other local stores |
| `python-local-api-test` | Local HTTP API probe + result report |

## Forbidden

- Writing to production or staging Firestore projects
- Starting the SDK client before exporting `FIRESTORE_EMULATOR_HOST` — the SDK connects to real Firestore silently
- Pasting service-account private key PEM material into skills, chat, or reports

## References

Read `references/reference.md` for: Go/Node/Java SDK env setup, seed JSON template, REST API for the emulator, and cleanup commands.
