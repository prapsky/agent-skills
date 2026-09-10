---
name: local-docker-dynamodb
description: >-
  Start, health-check, create tables, seed items, verify, and inspect local
  DynamoDB (Docker) for API and integration tests. Use when the user wants local
  DynamoDB, Docker Dynamo seeding, ticket-scoped scans, or preflight before
  local API tests that call Dynamo. Always use the local endpoint. Never use
  real AWS DynamoDB unless the user explicitly overrides.
---

# Local Docker DynamoDB

## Token rules (do these first)

1. **Local endpoint only** — default `http://127.0.0.1:8000` via container `app-dynamodb-local`.
2. Every AWS CLI/SDK call must include the local endpoint (or `DYNAMODB_ENDPOINT` / equivalent).
3. **Do not** print AWS access keys in chat or reports; dummy local keys are fine.
4. Create tables from the **project’s real key schema** — do not invent attributes that diverge from code.
5. **Phones / string IDs** in items → [`test-data-conventions`](../test-data-conventions/SKILL.md).
6. Long recipes → [reference.md](reference.md).

## When to use / skip

| Use | Skip |
|-----|------|
| Start / list-tables / seed / scan local Dynamo | Cases that never touch Dynamo |
| Ticket-scoped filter/scan for a phone/id | Real AWS unless user overrides |
| Preflight for services with `DYNAMODB_ENDPOINT` | MySQL/Redis/Firestore → sibling skills |

## Defaults (rename per project)

| Item | Default |
|------|---------|
| Container | `app-dynamodb-local` |
| Port | `8000` |
| Endpoint | `http://127.0.0.1:8000` |
| Region | `us-east-1` (or the project’s usual region — Local ignores it) |
| Image flags | `-sharedDb -inMemory` (data lost if container recreated) |

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. Start/verify app-dynamodb-local (list-tables)
- [ ] 3. Set DYNAMODB_ENDPOINT (or SDK endpoint_url) to http://127.0.0.1:8000
- [ ] 4. Create missing tables matching code models
- [ ] 5. PutItem / BatchWrite for the case + verify scan/query
- [ ] 6. Note stores_seeded in /tmp/<case>-seed.json
```

## Start + health

```bash
docker start app-dynamodb-local 2>/dev/null || \
docker run -d --name app-dynamodb-local -p 8000:8000 \
  amazon/dynamodb-local -jar DynamoDBLocal.jar -sharedDb -inMemory

export AWS_ACCESS_KEY_ID=local AWS_SECRET_ACCESS_KEY=local AWS_DEFAULT_REGION=us-east-1
aws dynamodb list-tables --endpoint-url http://127.0.0.1:8000 --region us-east-1
```

## Seed + verify (summary)

```bash
EP=(--endpoint-url http://127.0.0.1:8000 --region us-east-1)

aws dynamodb describe-table --table-name <TableName> "${EP[@]}"
aws dynamodb put-item --table-name <TableName> --item file:///tmp/<case>-dynamo-item.json "${EP[@]}"
aws dynamodb scan --table-name <TableName> \
  --filter-expression "contains(<attr>, :v)" \
  --expression-attribute-values '{":v":{"S":"<ticket-or-phone>"}}' \
  "${EP[@]}"
```

## Env alignment

| Pattern | Local value |
|---------|-------------|
| `DYNAMODB_ENDPOINT` / `AWS_ENDPOINT_URL_DYNAMODB` | `http://127.0.0.1:8000` |
| Empty endpoint | Usually means **real AWS** — forbidden for local tests |

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Phone `+6285YYMMDDxxx` + UUID string IDs |
| `local-docker-mysql` / `local-docker-redis` | Common API companions |
| `local-docker-firestore` | Document store emulator |
| `python-local-api-test` | Local API + result report |

## Forbidden (unless user explicitly overrides)

- AWS calls without the local endpoint
- Writing to shared staging/prod Dynamo tables
- Hard-coding one company’s table names as universal defaults in reports
