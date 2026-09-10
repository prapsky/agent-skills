# Local Docker Redis — reference

## Container

| Item | Default |
|------|---------|
| Name | `app-redis-local` |
| Image | `redis:7-alpine` |
| Port | `6379` |
| Auth | none (add `--requirepass` only if the app requires it locally) |

## Start

```bash
docker start app-redis-local 2>/dev/null || \
docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine
```

With password (optional):

```bash
docker run -d --name app-redis-local -p 6379:6379 redis:7-alpine \
  redis-server --requirepass 'local_only_password'
```

## Health + inspect

```bash
docker exec app-redis-local redis-cli ping
docker exec app-redis-local redis-cli DBSIZE
docker exec app-redis-local redis-cli INFO keyspace
docker exec app-redis-local redis-cli --scan --pattern '*'
```

### Typed gets

```bash
docker exec app-redis-local redis-cli TYPE '<key>'
docker exec app-redis-local redis-cli GET '<key>'      # string
docker exec app-redis-local redis-cli HGETALL '<key>'  # hash
docker exec app-redis-local redis-cli LRANGE '<key>' 0 -1  # list
docker exec app-redis-local redis-cli ZRANGE '<key>' 0 -1 WITHSCORES  # zset
```

## Env alignment

| Pattern | Local value |
|---------|-------------|
| `REDIS_HOST` | `127.0.0.1` |
| `REDIS_PORT` | `6379` |
| `REDIS_PASSWORD` | empty unless you set `--requirepass` |
| `REDIS_DB` / `REDIS_DB_ID` | usually `0` |

## Seed JSON note

When Redis is part of a multi-store case, record in `/tmp/<case>-seed.json`:

```json
{
  "stores_seeded": ["redis"],
  "redis_key_prefix": "seed:<case>:"
}
```

## Related

- Sibling skills: `local-docker-mysql`, `local-docker-dynamodb`, `local-docker-firestore`
- Often paired with `playwright-local-api-test`
