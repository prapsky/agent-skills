# Local Docker Pub/Sub — reference

## Container

| Item | Default |
|------|---------|
| Name | `app-pubsub-local` |
| Image | `gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators` |
| Port | `8085` |
| Command | `gcloud beta emulators pubsub start --host-port=0.0.0.0:8085` |

## Start

```bash
docker start app-pubsub-local 2>/dev/null || \
docker run -d --name app-pubsub-local -p 8085:8085 \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators \
  gcloud beta emulators pubsub start --host-port=0.0.0.0:8085
```

## Env patterns

| Var | Local value |
|-----|-------------|
| `PUBSUB_EMULATOR_HOST` | `127.0.0.1:8085` |
| `PUBSUB_PROJECT_ID` | `local-test` (rename per project) |
| `PUBSUB_*_TOPIC` | Keep topic **names** from project config |

Discover topic/subscription names from env files, infra config, or publisher code — do not guess.

## Inspect helpers

```bash
export PUBSUB_EMULATOR_HOST=127.0.0.1:8085
PROJECT=local-test

gcloud pubsub topics list --project="$PROJECT"
gcloud pubsub subscriptions list --project="$PROJECT"
gcloud pubsub topics describe <TopicName> --project="$PROJECT"
```

REST fallback:

```bash
curl -s "http://127.0.0.1:8085/v1/projects/local-test/topics"
```

## Push subscriber probe (HTTP handler)

When a service exposes a Pub/Sub **push** endpoint, POST the standard envelope:

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

Prefer emulator publish → pull/subscriber when testing full wiring.

## Cleanup

```bash
export PUBSUB_EMULATOR_HOST=127.0.0.1:8085
gcloud pubsub subscriptions delete "seed:<case>-sub" --project=local-test --quiet
docker rm -f app-pubsub-local   # drops in-memory topics
```

## Limitations

- Emulator ≠ production (ordering, DLQ, IAM, throughput).
- In-memory — recreate container ⇒ re-bootstrap topics.
- Good for wiring + payload shape; staging still needed for prod-like behavior.

## Related

- Sibling: `local-docker-mysql`, `local-docker-redis`, `local-docker-dynamodb`, `local-docker-firestore`
