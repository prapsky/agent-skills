---
name: local-docker-pubsub
description: >-
  Start, health-check, create topics/subscriptions, publish, pull, and inspect
  local Google Cloud Pub/Sub emulator (Docker) for API and integration tests.
  Use when the user wants local Pub/Sub, event publish/subscribe tests,
  ticket-scoped message checks, or preflight before local API tests that emit
  events. Set PUBSUB_EMULATOR_HOST to the local port. Never use real GCP Pub/Sub
  unless the user explicitly overrides.
---

# Local Docker Pub/Sub

## Token rules (do these first)

1. **Local emulator only** — default container `app-pubsub-local` on `127.0.0.1:8085`.
2. Set `PUBSUB_EMULATOR_HOST=127.0.0.1:8085` for every `gcloud pubsub` call and for services under test.
3. **Do not** print GCP service-account JSON or production project secrets in chat or reports.
4. Create topics/subscriptions from the **project’s env / code** — do not invent names that diverge from the codebase.
5. Tag test payloads with the ticket/case id (`seed:<ticket>` in JSON or message attributes).
6. Long recipes → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Start / list topics / publish / pull on local emulator | Flows with no Pub/Sub |
| Ticket-scoped verify after API publish | Real GCP unless user overrides |
| Preflight for services reading `PUBSUB_*` env | MySQL/Redis/Dynamo/Firestore → sibling skills |

## Defaults (rename per project)

| Item | Default |
|------|---------|
| Container | `app-pubsub-local` |
| Port | `8085` |
| Emulator host | `127.0.0.1:8085` |
| Local project ID | `local-test` (any string works with emulator) |
| Image | `gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators` |

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. Start/verify app-pubsub-local on :8085
- [ ] 3. export PUBSUB_EMULATOR_HOST=127.0.0.1:8085
- [ ] 4. Point PUBSUB_PROJECT_ID (and topic env vars) at local project + real topic names from code
- [ ] 5. Create topics + subscriptions the case needs
- [ ] 6. Publish seed message or trigger API → pull / inspect subscription
- [ ] 7. Note pubsub_seeded in /tmp/<case>-seed.json when continuing to API tests
```

## Start + health

```bash
docker start app-pubsub-local 2>/dev/null || \
docker run -d --name app-pubsub-local -p 8085:8085 \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators \
  gcloud beta emulators pubsub start --host-port=0.0.0.0:8085

export PUBSUB_EMULATOR_HOST=127.0.0.1:8085
gcloud pubsub topics list --project=local-test
# empty list is OK on fresh emulator — still confirms emulator is up
```

## Bootstrap + verify (summary)

```bash
export PUBSUB_EMULATOR_HOST=127.0.0.1:8085
PROJECT=local-test
TOPIC=<TopicNameFromEnvOrCode>

gcloud pubsub topics create "$TOPIC" --project="$PROJECT" 2>/dev/null || true
gcloud pubsub subscriptions create "seed:<TICKET>-sub" \
  --topic="$TOPIC" --project="$PROJECT" 2>/dev/null || true

gcloud pubsub topics publish "$TOPIC" --project="$PROJECT" \
  --message='{"seed":"<TICKET>"}'

gcloud pubsub subscriptions pull "seed:<TICKET>-sub" --project="$PROJECT" \
  --limit=5 --auto-ack
```

## Env alignment

| Pattern | Local value |
|---------|-------------|
| `PUBSUB_EMULATOR_HOST` | `127.0.0.1:8085` (shell + process env) |
| `PUBSUB_PROJECT_ID` | `local-test` (or project-specific local id) |
| Topic env vars | Keep **names** from project config; only project id + emulator host change |

Go/Java/Node GCP Pub/Sub clients honor `PUBSUB_EMULATOR_HOST` automatically.

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Ticket/phone tags inside event payloads |
| `local-docker-mysql` / `local-docker-redis` / `local-docker-dynamodb` / `local-docker-firestore` | Common API companions |
| `python-local-api-test` | Local API + result report |

## Forbidden (unless user explicitly overrides)

- Publishing to shared staging/prod GCP projects without emulator host
- Omitting `PUBSUB_EMULATOR_HOST` while claiming “local Pub/Sub test”
- Hard-coding one company’s topic list as universal defaults in reports

Details: [reference.md](reference.md).
