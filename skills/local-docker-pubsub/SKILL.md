---
name: local-docker-pubsub
description: >-
  Use this skill whenever you need to start, create topics/subscriptions,
  publish, pull, or inspect the local Google Cloud Pub/Sub emulator for API and
  integration tests. Triggers on: "local Pub/Sub", "PUBSUB_EMULATOR_HOST",
  "publish event locally", "pull messages locally", "seed Pub/Sub topic",
  "emulator pubsub", "event-driven local test", or any request to test
  publish/subscribe wiring before a local HTTP test. Always set
  PUBSUB_EMULATOR_HOST — an empty host sends traffic to real GCP Pub/Sub
  (forbidden unless the user explicitly overrides).
---

# Local Docker Pub/Sub

Start, bootstrap, publish, and inspect the local GCP Pub/Sub emulator for
API and integration tests. Setting `PUBSUB_EMULATOR_HOST` is what routes all
gcloud and SDK calls to the local emulator instead of real GCP.

## When to use

| Use this skill | Skip and use instead |
|---------------|----------------------|
| Start / list topics / publish / pull on local emulator | Flows with no Pub/Sub events |
| Ticket-scoped message verify after API publish | Real GCP Pub/Sub — only if user overrides |
| Preflight for services that read `PUBSUB_EMULATOR_HOST` | MySQL/Redis/Dynamo/Firestore → sibling skills |

## Do these first (token rules)

1. Export `PUBSUB_EMULATOR_HOST=127.0.0.1:8085` for every `gcloud pubsub` call and for the service under test — the SDK routes to the emulator only when this env var is set.
2. Discover topic and subscription names from the project's env files, infra config, or publisher code — do not guess or invent names.
3. Do not print GCP service-account JSON or production project secrets in chat or reports.
4. Tag test message payloads with the ticket/case id (`"seed":"<TICKET>"`) so you can trace messages after publish.

## Defaults

| Item | Default |
|------|---------|
| Container | `app-pubsub-local` |
| Port | `8085` |
| Emulator host env | `PUBSUB_EMULATOR_HOST=127.0.0.1:8085` |
| Local project ID | `local-test` (any string works with the emulator) |
| Image | `gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators` |

## Workflow

1. **Start the container** (idempotent):

   ```bash
   docker start app-pubsub-local 2>/dev/null || \
   docker run -d --name app-pubsub-local -p 8085:8085 \
     gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators \
     gcloud beta emulators pubsub start --host-port=0.0.0.0:8085
   ```

2. **Export the emulator host and verify:**

   ```bash
   export PUBSUB_EMULATOR_HOST=127.0.0.1:8085
   gcloud pubsub topics list --project=local-test
   # empty list is OK on a fresh emulator — confirms the emulator is reachable
   ```

3. **Align the app's local env:**

   | Pattern | Local value |
   |---------|-------------|
   | `PUBSUB_EMULATOR_HOST` | `127.0.0.1:8085` (shell + process env) |
   | `PUBSUB_PROJECT_ID` | `local-test` (or project-specific local id) |
   | Topic env vars | Keep **names** from project config; only project id + emulator host change |

   Go/Java/Node GCP Pub/Sub clients honour `PUBSUB_EMULATOR_HOST` automatically.

4. **Bootstrap topics and subscriptions** (names come from project config):

   ```bash
   PROJECT=local-test
   TOPIC=<TopicNameFromEnvOrCode>

   gcloud pubsub topics create "$TOPIC" --project="$PROJECT" 2>/dev/null || true
   gcloud pubsub subscriptions create "seed:<TICKET>-sub" \
     --topic="$TOPIC" --project="$PROJECT" 2>/dev/null || true
   ```

5. **Publish a seed message** and pull to verify delivery:

   ```bash
   gcloud pubsub topics publish "$TOPIC" --project="$PROJECT" \
     --message='{"seed":"<TICKET>"}'

   gcloud pubsub subscriptions pull "seed:<TICKET>-sub" \
     --project="$PROJECT" --limit=5 --auto-ack
   ```

6. **Push subscriber probe** — when the service exposes a push endpoint, POST the standard envelope:

   ```bash
   curl -s -X POST "http://127.0.0.1:<PORT>/<push-path>" \
     -H 'Content-Type: application/json' \
     -d '{
       "message": {
         "data": "'$(echo -n '{"seed":"<TICKET>"}' | base64)'",
         "messageId": "seed-<TICKET>-1",
         "publishTime": "2026-01-01T00:00:00Z"
       },
       "subscription": "projects/local-test/subscriptions/<SubscriptionName>"
     }'
   ```

7. **Record in `/tmp/<case>-seed.json`:**

   ```json
   {
     "stores_seeded": ["pubsub"],
     "pubsub_emulator": "127.0.0.1:8085",
     "pubsub_topic": "<TopicName>",
     "pubsub_subscription": "seed:<TICKET>-sub"
   }
   ```

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. app-pubsub-local healthy on 127.0.0.1:8085
- [ ] 3. PUBSUB_EMULATOR_HOST=127.0.0.1:8085 exported for app process
- [ ] 4. Topic names discovered from project config (not invented)
- [ ] 5. Topic + subscription created
- [ ] 6. Seed message published + pull confirmed (or push probe sent)
- [ ] 7. /tmp/<case>-seed.json written
```

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Ticket/phone tags inside event payloads |
| `local-docker-mysql` / `local-docker-redis` / `local-docker-dynamodb` / `local-docker-firestore` | Common API companions |
| `python-local-api-test` | Local HTTP API probe + result report |

## Forbidden

- Publishing to shared staging or production GCP projects without `PUBSUB_EMULATOR_HOST` set — messages reach real subscribers
- Omitting `PUBSUB_EMULATOR_HOST` while claiming a "local Pub/Sub test"
- Hard-coding a specific project's topic list as universal defaults in this skill

## References

Read `references/reference.md` for: REST fallback for topic listing, push-subscriber envelope format, cleanup commands, and emulator limitations vs production.
