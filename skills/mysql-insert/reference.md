# MySQL insert — reference

## Local Docker stack (MySQL + Redis)

**Never** use a cloud DB proxy, remote staging MySQL, or remote staging Redis for these flows.

### MySQL — `app-mysql-local`

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

| Item | Default value |
|------|----------------|
| Host / port | `127.0.0.1:3306` |
| Database | `app_local` |
| User / password | `app` / `app_local` |
| Root password | `app_root_local` (container admin only) |

If schema/FK rows are missing, load a local dump or create minimal lookup rows before seeding.

Projects may rename containers/creds — keep Docker and `.env*` in sync.

### Redis — `app-redis-local`

Required when the API / Playwright run needs cache (common for local API tests).

```bash
docker start app-redis-local 2>/dev/null || \
docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine

docker exec app-redis-local redis-cli ping
# expect: PONG
```

| Item | Default value |
|------|----------------|
| Host / port | `127.0.0.1:6379` |
| Password | none (`REDIS_PASSWORD=` empty) |

### Local env alignment

Point the service’s local env (`.env.testing`, `.env.local`, etc.) at Docker — do **not** switch back to remote staging for seed/test runs:

| Var pattern | Local value |
|-------------|-------------|
| DB host | `127.0.0.1` |
| MySQL port | `3306` |
| MySQL name / user / password | match Docker `MYSQL_*` |
| `REDIS_HOST` / `REDIS_PORT` | `127.0.0.1` / `6379` |
| `REDIS_PASSWORD` | empty (unless you set `--requirepass`) |

If the app defaults to a cloud connector, force plain TCP to localhost (exact env names vary by project), for example:

- `DATABASE_INSTANCE_CONNECTION_TYPE=tcp` (or the project’s equivalent)
- Clear / ignore cloud instance connection names for the local run

Do **not** print passwords in chat or reports.

## DBeaver

- One statement at a time (Ctrl+Enter).
- Avoid `SET @var` multi-scripts unless script mode is confirmed.
- Connect to Docker (`127.0.0.1:3306`), not remote staging.

## Example `INSERT … SELECT` pattern

Replace table/column/name literals with the project’s real schema:

```sql
INSERT INTO <table> (<cols…>)
SELECT
  '<seed-id>', …, parent.id, …
FROM <fk_table> parent
WHERE parent.<unique_name_col> = '<known-fixture-name>'
LIMIT 1;
```

## Verify / cleanup pattern

```sql
SELECT <key_cols> FROM <table> WHERE <id_or_tag> = '<seed-id>';
DELETE FROM <table> WHERE <tag_col> LIKE 'seed:%';  -- only if user asks
```

## pymysql (when CLI auth plugin fails)

```bash
python3 -m venv /tmp/seed-venv && /tmp/seed-venv/bin/pip install -q pymysql
# connect host=127.0.0.1 port=3306 user/password/db matching Docker MYSQL_*
```

Never print DB passwords. Write seed to `/tmp/<case>-seed.json` for Playwright.
