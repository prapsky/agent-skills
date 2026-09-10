# Playwright local API — reference

## Local Docker stack (hard rule)

| Do | Do not |
|----|--------|
| Docker MySQL `app-mysql-local` on `127.0.0.1:3306` | Cloud DB proxy to remote staging |
| Docker Redis `app-redis-local` on `127.0.0.1:6379` | Remote staging Redis hosts / passwords |
| Seed via `local-docker-mysql` against Docker MySQL | Seeding or testing against remote staging DBs |
| Load local env aligned to Docker | Tunnel ports used only to reach remote DBs |

### Start MySQL

```bash
docker start app-mysql-local 2>/dev/null || \
docker run -d --name app-mysql-local \
  -e MYSQL_DATABASE=app_local \
  -e MYSQL_USER=app \
  -e MYSQL_PASSWORD=app_local \
  -e MYSQL_ROOT_PASSWORD=app_root_local \
  -p 3306:3306 \
  mysql:8.0 \
  --default-authentication-plugin=mysql_native_password

docker exec app-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -papp_root_local --silent
```

### Start Redis

```bash
docker start app-redis-local 2>/dev/null || \
docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine

docker exec app-redis-local redis-cli ping
# expect: PONG
```

### Env

| Concern | Local value |
|---------|-------------|
| MySQL | TCP → `127.0.0.1:3306` / match Docker `MYSQL_*` |
| Redis | `127.0.0.1:6379`, empty password by default |

Seed / more detail: `local-docker-mysql` skill + its `reference.md`.

## Result report template

Always use these headings after a local test run (chat + `result.html` + PR comment when a PR is in context):

```markdown
### The issue
...
**Issue log:**
```json
{
  "record_id": "…",
  "problem": "…"
}
```

### What is the root cause
...

### What you actually did in this PR
...

### The local endpoint URL
`http://127.0.0.1:<port>/<api-path>`

### The request body
```json
{
  "method": "GET",
  "path": "/<api-path>",
  "headers": {
    "Authorization": "Bearer <redacted>",
    "Content-Type": "application/json"
  },
  "body": null
}
```

### The response
```json
{
  "data": {
    "example_field": "readable value"
  },
  "meta": {
    "status_code": 200
  }
}
```

### The explanation of the result
One simple sentence about pass or fail.
```

`result.html` should also include: PASSED/FAILED banner, Docker MySQL/Redis note, ISO time, and a link to `./index.html`.

## Seed → body

Map fields from `/tmp/<case>-seed.json` to the API request body using the endpoint’s contract. Keep the mapping in the spec or `/tmp/<case>-body.json` — do not hard-code one product’s field names in this skill.

## Run

```bash
cd "<repo>/playwright/local-api"   # or the repo’s existing Playwright API folder
PW_BASE_URL='http://127.0.0.1:<port>' \
PW_PATH='/<api-path>' \
PW_SEED_JSON="$(cat /tmp/<case>-seed.json)" \
PW_BODY_JSON="$(cat /tmp/<case>-body.json)" \
npx playwright test tests/<spec>.ts --reporter=list,html
```

## Pairing with `local-docker-mysql`

1. Docker daemon up  
2. `app-mysql-local` + `app-redis-local` healthy  
3. Seed + `/tmp/<case>-seed.json`  
4. Map body → `/tmp/<case>-body.json`  
5. Start API once against local Docker; run Playwright **or** curl — not both without reseed when the first call consumes the fixture  
6. Always deliver the Result report (chat + `result.html`; auto PR comment when PR in context)  

## Cleanup

Only if user asks: delete rows tagged as seeds (project-specific SQL).
