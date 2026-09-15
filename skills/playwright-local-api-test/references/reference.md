# Playwright local UI test — reference

> **API testing?** Use `python-local-api-test` — not Playwright.
> This reference covers Docker stack setup for UI tests only.

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

### Env alignment

| Concern | Local value |
|---------|-------------|
| MySQL | TCP → `127.0.0.1:3306` / match Docker `MYSQL_*` |
| Redis | `127.0.0.1:6379`, empty password by default |

Seed details: see `local-docker-mysql` skill and its `references/reference.md`.

## Run a Playwright spec

```bash
cd "<repo>/playwright"
PW_BASE_URL='http://127.0.0.1:<port>' \
npx playwright test tests/<spec>.ts --reporter=list,html
```

## Pairing with seeded data

1. Docker daemon up
2. `app-mysql-local` + `app-redis-local` healthy
3. Seed + write `/tmp/<case>-seed.json`
4. Start the UI server against local Docker
5. Run Playwright spec — do **not** reseed mid-run when a fixture is consumed by the first interaction

## Cleanup

Only if user asks: delete rows tagged as seeds (project-specific SQL) or flush seed-prefixed Redis keys.

## Related skills

- `local-docker-mysql` — full MySQL seed recipes and reference
- `local-docker-redis` — Redis seed recipes and reference
- `python-local-api-test` — use this for any HTTP API probe or result report
- `test-data-conventions` — phone and UUID conventions for UI form inputs
