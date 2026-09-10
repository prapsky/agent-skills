# Playwright local API — reference

## Local Docker stack (hard rule)

| Do | Do not |
|----|--------|
| Docker MySQL `fithub-mysql-local` on `127.0.0.1:3306` | `~/bin/cloud-sql-proxy` |
| Docker Redis `fithub-redis-local` on `127.0.0.1:6379` | Staging Redis IPs / passwords |
| Seed via `mysql-insert` against Docker MySQL | Staging Cloud SQL (`fit-hub-staging:…`) |
| Load `.env.testing` already aligned to local Docker | Port **3307** proxy to staging |

### Start MySQL

```bash
docker start fithub-mysql-local 2>/dev/null || \
docker run -d --name fithub-mysql-local \
  -e MYSQL_DATABASE=fithub-local \
  -e MYSQL_USER=fithub \
  -e MYSQL_PASSWORD=fithub_local \
  -e MYSQL_ROOT_PASSWORD=fithub_root_local \
  -p 3306:3306 \
  mysql:8.0 \
  --default-authentication-plugin=mysql_native_password

docker exec fithub-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -pfithub_root_local --silent
```

### Start Redis

```bash
docker start fithub-redis-local 2>/dev/null || \
docker run -d --name fithub-redis-local -p 6379:6379 redis:7-alpine

docker exec fithub-redis-local redis-cli ping
# expect: PONG
```

### Env (already in `.env.testing`)

| Service | MySQL | Redis |
|---------|-------|-------|
| BACKEND-GOLANG | TCP + whitelist → `127.0.0.1:3306` / `fithub-local` | `127.0.0.1:6379`, empty password |
| free-trial-service | `CONNECTION_TYPE=tcp` → same Docker MySQL | same |
| lms-service | master `CONNECTION_TYPE=private` → `@tcp` same Docker MySQL | same |

Seed / more detail: `mysql-insert` skill + its `reference.md`.

## `result.html` (minimal)

Include: PASSED/FAILED, endpoint, seed (no secrets), request, status, response, one-line explanation, link to `./index.html`, ISO time.

## Seed → body (free-trial)

| Seed | Body |
|------|------|
| `club` | `locationUser` |
| `phone` / `email` / `leadsName` | same fields |
| omit `dealDate` | exercises empty Dynamo `dealDate` path |

## Run

```bash
cd "BE - BACKEND-GOLANG/playwright/local-api"
PW_BASE_URL='http://127.0.0.1:5005' \
PW_PATH='/v1/leads/free-trial' \
PW_SEED_JSON="$(cat /tmp/<case>-seed.json)" \
PW_BODY_JSON="$(cat /tmp/<case>-body.json)" \
npx playwright test tests/<spec>.ts --reporter=list,html
```

## Pairing with `mysql-insert`

1. Docker daemon up  
2. `fithub-mysql-local` + `fithub-redis-local` healthy  
3. Seed + `/tmp/<case>-seed.json`  
4. Map body → `/tmp/<case>-body.json`  
5. Start API once against local Docker; run Playwright **or** curl — not both without reseed  
6. `result.html` + optional PR comment  

## Cleanup

Only if user asks: `DELETE FROM leads WHERE remarks LIKE 'seed:%';`
