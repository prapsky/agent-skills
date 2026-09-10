# Local Docker MySQL — reference

## Container

| Item | Default |
|------|---------|
| Name | `app-mysql-local` |
| Image | `mysql:8.0` |
| Port | `3306` |
| Auth | `mysql_native_password` (friendlier for pymysql / older clients) |
| Database | `app_local` |
| User / password | `app` / `app_local` |
| Root password | `app_root_local` (container admin only) |

Projects may rename the container or credentials — keep Docker `MYSQL_*` and the app’s `.env.testing` / `.env.local` identical.

**Never** use a cloud DB proxy or remote staging MySQL for these flows.

## Start

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

## Health + list

```bash
docker exec app-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -papp_root_local --silent
docker exec app-mysql-local mysql -uapp -papp_local -e "SHOW DATABASES;"
docker exec app-mysql-local mysql -uapp -papp_local app_local -e "SHOW TABLES;"
```

## Redis companion (API / Playwright)

When the API needs cache, use `local-docker-redis` (`app-redis-local` on `6379`). Quick ping:

```bash
docker start app-redis-local 2>/dev/null || \
docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine
docker exec app-redis-local redis-cli ping
# expect: PONG
```

## Local env alignment

| Var pattern | Local value |
|-------------|-------------|
| DB host | `127.0.0.1` |
| MySQL port | `3306` |
| MySQL name / user / password | match Docker `MYSQL_*` |
| `REDIS_HOST` / `REDIS_PORT` | `127.0.0.1` / `6379` when Redis is used |
| `REDIS_PASSWORD` | empty (unless you set `--requirepass`) |
| Cloud connector | force plain TCP / clear instance connection name |

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
# host=127.0.0.1 port=3306 user=app password=app_local database=app_local
```

Never print DB passwords. Write seed to `/tmp/<case>-seed.json` for Playwright.

## Dump / restore (optional)

```bash
docker exec app-mysql-local mysqldump -uapp -papp_local app_local > /tmp/app_local-dump.sql
docker exec -i app-mysql-local mysql -uapp -papp_local app_local < /tmp/app_local-dump.sql
```

## Empty DB

If there are no tables, apply a **minimal** schema for the case or load a local dump — do not invent production-like UUIDs for FKs; discover or insert lookup rows first.

## Related

- Sibling skills: `local-docker-redis`, `local-docker-dynamodb`, `local-docker-firestore`
- API reporting: `playwright-local-api-test`
