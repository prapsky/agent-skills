---
name: local-docker-dynamodb
description: >-
  Use this skill whenever you need to start, create tables, seed items, verify,
  or inspect local DynamoDB (Docker) for API and integration tests. Triggers on:
  "seed local DynamoDB", "local Dynamo table", "PutItem locally", "DynamoDB
  emulator", "list-tables local", "ticket-scoped Dynamo scan", or any request
  to prepare DynamoDB state before a local HTTP test. Always use the local
  endpoint — never call real AWS DynamoDB unless the user explicitly overrides.
---

# Local Docker DynamoDB

Start, create tables, seed items, verify, and inspect the local DynamoDB
container for API and integration tests. The local endpoint keeps every write
away from shared AWS tables in staging.

## When to use

| Use this skill | Skip and use instead |
|---------------|----------------------|
| Start / list-tables / seed / scan local Dynamo | Cases that never touch DynamoDB |
| Ticket-scoped filter/scan by phone or id | Real AWS DynamoDB — only if user overrides |
| Preflight for services with `DYNAMODB_ENDPOINT` | MySQL/Redis/Firestore → sibling skills |

## Do these first (token rules)

1. Every AWS CLI/SDK call must include the local endpoint — an empty endpoint usually means **real AWS** (forbidden for local tests).
2. Create tables from the **project's real key schema** — do not invent attributes that diverge from code.
3. Do not print AWS access keys in chat or reports; dummy local credentials (`local`/`local`) are intentionally non-secret.
4. Phones and string IDs in items → follow [`test-data-conventions`](../test-data-conventions/SKILL.md).
5. The `-inMemory` flag loses data when the container is removed — re-bootstrap tables when that happens.

## Defaults

| Item | Default |
|------|---------|
| Container | `app-dynamodb-local` |
| Port | `8000` |
| Endpoint | `http://127.0.0.1:8000` |
| Region | `us-east-1` (local ignores it but must be set) |
| Access key / secret | `local` / `local` |
| Image flags | `-sharedDb -inMemory` |

## Workflow

1. **Start the container** (idempotent):

   ```bash
   docker start app-dynamodb-local 2>/dev/null || \
   docker run -d --name app-dynamodb-local -p 8000:8000 \
     amazon/dynamodb-local -jar DynamoDBLocal.jar -sharedDb -inMemory
   ```

2. **Set local credentials + verify:**

   ```bash
   export AWS_ACCESS_KEY_ID=local AWS_SECRET_ACCESS_KEY=local AWS_DEFAULT_REGION=us-east-1
   aws dynamodb list-tables --endpoint-url http://127.0.0.1:8000 --region us-east-1
   # empty list is OK on a fresh container — confirms emulator is reachable
   ```

3. **Set the endpoint alias** for repeated commands:

   ```bash
   EP=(--endpoint-url http://127.0.0.1:8000 --region us-east-1)
   ```

4. **Create missing tables** — copy key/attribute names from the project's service model:

   ```bash
   aws dynamodb create-table \
     --table-name <TableName> \
     --attribute-definitions AttributeName=<pk>,AttributeType=S \
     --key-schema AttributeName=<pk>,KeyType=HASH \
     --billing-mode PAY_PER_REQUEST \
     "${EP[@]}"
   ```

   Add GSIs only when the service code queries them.

5. **Seed an item:**

   ```bash
   aws dynamodb put-item --table-name <TableName> \
     --item file:///tmp/<case>-dynamo-item.json \
     "${EP[@]}"
   ```

6. **Verify** with a filter scan scoped to the ticket:

   ```bash
   aws dynamodb scan --table-name <TableName> \
     --filter-expression "contains(#attr, :v)" \
     --expression-attribute-names '{"#attr":"<field>"}' \
     --expression-attribute-values '{":v":{"S":"seed:<TICKET>"}}' \
     "${EP[@]}"
   ```

7. **Set the endpoint env** for the API process so SDK calls hit local Dynamo:

   ```bash
   export DYNAMODB_ENDPOINT=http://127.0.0.1:8000
   # or: AWS_ENDPOINT_URL_DYNAMODB=http://127.0.0.1:8000
   ```

8. **Record in `/tmp/<case>-seed.json`:**

   ```json
   {
     "stores_seeded": ["dynamodb"],
     "dynamo_endpoint": "http://127.0.0.1:8000",
     "tables": ["<TableName>"]
   }
   ```

## Checklist

```
- [ ] 1. Docker daemon up
- [ ] 2. app-dynamodb-local healthy (list-tables) on http://127.0.0.1:8000
- [ ] 3. DYNAMODB_ENDPOINT set to http://127.0.0.1:8000 for app process
- [ ] 4. Tables created matching project key schema
- [ ] 5. PutItem / BatchWrite succeeded + filter scan confirms item
- [ ] 6. /tmp/<case>-seed.json written
```

## Examples

**Item JSON for `PutItem`** (`/tmp/<case>-dynamo-item.json`):

```json
{
  "<pk>": { "S": "a3f1c2e4-9b8d-4e2a-91f0-7c6d5b4a3210" },
  "phone": { "S": "+6285260915001" },
  "tag":   { "S": "seed:acq-2937" }
}
```

**Get a single item:**

```bash
aws dynamodb get-item --table-name <TableName> \
  --key '{"<pk>":{"S":"a3f1c2e4-9b8d-4e2a-91f0-7c6d5b4a3210"}}' \
  "${EP[@]}"
```

## Related skills

| Skill | Role |
|-------|------|
| `test-data-conventions` | Phone `+6285YYMMDDxxx` + UUID string IDs |
| `local-docker-mysql` / `local-docker-redis` | Common API companions |
| `local-docker-firestore` | Document store emulator |
| `python-local-api-test` | Local HTTP API probe + result report |

## Forbidden

- AWS CLI/SDK calls without `--endpoint-url http://127.0.0.1:8000` — an empty endpoint hits real AWS
- Writing to shared staging or production DynamoDB tables
- Hard-coding a specific project's table names as universal defaults in this skill

## References

Read `references/reference.md` for: BatchWrite syntax, GSI creation templates, SDK endpoint config, env alignment table, and durable volume setup (when `-inMemory` is not acceptable).
