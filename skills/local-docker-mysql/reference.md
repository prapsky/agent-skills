# Local Docker MySQL — reference

## Container

| Item | Default |
|------|---------|
| Name | `app-mysql-local` |
| Image | `mysql:8.0` |
| Port | `3306` |
| Auth | `mysql_native_password` (friendlier for pymysql / older clients) |

Projects may rename the container or credentials — keep Docker `MYSQL_*` and the app’s `.env.testing` / `.env.local` identical.

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
```

## Health + list

```bash
docker exec app-mysql-local mysqladmin ping -h 127.0.0.1 -uroot -papp_root_local --silent
docker exec app-mysql-local mysql -uapp -papp_local -e "SHOW DATABASES;"
docker exec app-mysql-local mysql -uapp -papp_local app_local -e "SHOW TABLES;"
```

## pymysql

```bash
python3 -m venv /tmp/seed-venv && /tmp/seed-venv/bin/pip install -q pymysql
# host=127.0.0.1 port=3306 user=app password=app_local database=app_local
```

## Env alignment (generic)

| Pattern | Local value |
|---------|-------------|
| DB host | `127.0.0.1` |
| Port | `3306` |
| Name / user / password | match Docker `MYSQL_*` |
| Cloud connector | force plain TCP / clear instance connection name |

Do not print passwords in chat or reports.

## Dump / restore (optional)

```bash
docker exec app-mysql-local mysqldump -uapp -papp_local app_local > /tmp/app_local-dump.sql
docker exec -i app-mysql-local mysql -uapp -papp_local app_local < /tmp/app_local-dump.sql
```

## Empty DB

If there are no tables, apply a **minimal** schema for the case or load a local dump — do not invent production-like UUIDs for FKs; discover or insert lookup rows first.

## Related

- `mysql-insert` for INSERT…SELECT templates
- Sibling skills: `local-docker-redis`, `local-docker-dynamodb`, `local-docker-firestore`
