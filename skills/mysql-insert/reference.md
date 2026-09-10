# MySQL insert — reference

## Local Docker stack (MySQL + Redis)

**Never** use Cloud SQL proxy, staging MySQL, or staging Redis.

### MySQL — `fithub-mysql-local`

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

| Item | Value |
|------|--------|
| Host / port | `127.0.0.1:3306` |
| Database | `fithub-local` |
| User / password | `fithub` / `fithub_local` |
| Root password | `fithub_root_local` (container admin only) |

If schema/FK rows are missing, load a local dump or create minimal lookup rows before seeding leads.

### Redis — `fithub-redis-local`

Required when the API / Playwright run needs cache (almost always for local API tests).

```bash
docker start fithub-redis-local 2>/dev/null || \
docker run -d --name fithub-redis-local -p 6379:6379 redis:7-alpine

docker exec fithub-redis-local redis-cli ping
# expect: PONG
```

| Item | Value |
|------|--------|
| Host / port | `127.0.0.1:6379` |
| Password | none (`REDIS_PASSWORD=` empty) |

### `.env.testing` alignment

These repos already use the local Docker stack — do **not** switch them back to staging:

- `BE - BACKEND-GOLANG/.env.testing`
- `BE - free-trial-service/.env.testing`
- `BE - lms-service/.env.testing`

| Var pattern | Local value |
|-------------|-------------|
| `DATABASE_*` / `DATABASE_MASTER_*` host | `127.0.0.1` |
| MySQL port | `3306` |
| MySQL name / user | `fithub-local` / `fithub` |
| `REDIS_HOST` | `127.0.0.1` |
| `REDIS_PORT` | `6379` |
| `REDIS_PASSWORD` | empty |

BACKEND-GOLANG also needs TCP to Docker: `DATABASE_INSTANCE_CONNECTION_TYPE=tcp`, `WHITELIST_DATABASE_PRIVATE_CONNECTION_TYPE=.*`, `DATABASE_HOSTNAME_PRIVATE=127.0.0.1`.  
lms-service uses `DATABASE_MASTER_INSTANCE_CONNECTION_TYPE=private` for plain `@tcp` (existing code behavior).  
free-trial-service uses `DATABASE_INSTANCE_CONNECTION_TYPE=tcp` (TCP path in `pkg/mysql/setup.go`).

Do **not** print passwords in chat or reports.

## DBeaver

- One statement at a time (Ctrl+Enter).
- Avoid `SET @var` multi-scripts unless script mode is confirmed.
- Connect to Docker (`127.0.0.1:3306`), not staging.

## `INSERT … SELECT`

```sql
INSERT INTO leads (id, parent_id, club_id, phone, email, name, gender, status_id, source_id,
  preferred_contact_time_id, deal_at, registered_at, created_at, updated_at, created_by, updated_by, remarks)
SELECT
  'dealdate-seed-1', 'dealdate-seed-1', c.id, '62812…', 'seed@example.com', 'Seed',
  'Female', 1, s.id, pct.id, NULL, UNIX_TIMESTAMP(), UNIX_TIMESTAMP(), UNIX_TIMESTAMP(),
  'seed@fithub.id', 'seed@fithub.id', 'seed:dealdate-empty-gsi'
FROM clubs c
JOIN lead_sources s ON s.name = 'Website'
JOIN lead_preferred_contact_time pct ON pct.name = 'Anytime'
WHERE c.full_name = 'FIT HUB BENHIL'
LIMIT 1;
```

## Verify / cleanup

```sql
SELECT id, phone, club_id, deal_at, remarks FROM leads WHERE id = 'dealdate-seed-1';
DELETE FROM leads WHERE remarks LIKE 'seed:%';
```

## pymysql (when CLI auth plugin fails)

```bash
python3 -m venv /tmp/seed-venv && /tmp/seed-venv/bin/pip install -q pymysql
# connect host=127.0.0.1 port=3306 user=fithub password=fithub_local database=fithub-local
```

Never print `DATABASE_PASSWORD`. Write seed to `/tmp/<case>-seed.json` for Playwright.
