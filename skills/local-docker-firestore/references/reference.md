# Local Docker Firestore — reference

## Container

| Item | Default |
|------|---------|
| Name | `app-firestore-local` |
| Image | `gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators` |
| Port | `8080` |
| Cmd | `gcloud beta emulators firestore start --host-port=0.0.0.0:8080 --project=<project>` |

## Start (Docker)

```bash
docker start app-firestore-local 2>/dev/null || \
docker run -d --name app-firestore-local -p 8080:8080 \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:emulators \
  gcloud beta emulators firestore start --host-port=0.0.0.0:8080 --project=demo-project
```

## Start (host gcloud)

```bash
gcloud emulators firestore start --host-port=127.0.0.1:8080
```

## Health

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080
# Any HTTP response usually means the port is accepting connections
```

There is no universal SQL-like `SELECT *` — verify by reading known collection/document paths from the app.

## Env for every process under test

```bash
export FIRESTORE_EMULATOR_HOST=127.0.0.1:8080
```

Also set this in the service’s `.env.testing` / `.env.local` so child processes inherit it.

## Go / Node / Python

All official clients honor `FIRESTORE_EMULATOR_HOST` when set before client construction. Construct the client **after** exporting the env var.

## Seed JSON note

```json
{
  "stores_seeded": ["firestore"],
  "firestore_emulator": "127.0.0.1:8080",
  "firestore_paths": ["collection/doc"]
}
```

## Related

- Sibling skills: `local-docker-mysql`, `local-docker-redis`, `local-docker-dynamodb`
- Pair with `python-local-api-test` when exercising HTTP APIs that read Firestore
