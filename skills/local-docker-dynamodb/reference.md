# Local Docker DynamoDB — reference

## Container

| Item | Default |
|------|---------|
| Name | `app-dynamodb-local` |
| Image | `amazon/dynamodb-local` |
| Port | `8000` |
| Cmd | `-jar DynamoDBLocal.jar -sharedDb -inMemory` |

`-inMemory` loses data when the container is removed. Use a Docker volume + `-dbPath` if the project needs durable local tables.

## Start

```bash
docker start app-dynamodb-local 2>/dev/null || \
docker run -d --name app-dynamodb-local -p 8000:8000 \
  amazon/dynamodb-local -jar DynamoDBLocal.jar -sharedDb -inMemory
```

## AWS CLI helpers

```bash
export AWS_ACCESS_KEY_ID=local
export AWS_SECRET_ACCESS_KEY=local
export AWS_DEFAULT_REGION=us-east-1
EP=(--endpoint-url http://127.0.0.1:8000 --region us-east-1)

aws dynamodb list-tables "${EP[@]}"
aws dynamodb list-tables "${EP[@]}" --output text
```

## Create table (template only)

Copy attribute/key names from the service model — replace placeholders:

```bash
aws dynamodb create-table \
  --table-name <TableName> \
  --attribute-definitions \
    AttributeName=<pk>,AttributeType=S \
  --key-schema AttributeName=<pk>,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  "${EP[@]}"
```

Add GSIs only when the code queries them. Local Dynamo accepts provisioned or on-demand; prefer matching production-ish definitions from the project.

## Put / get / scan

```bash
aws dynamodb put-item --table-name <TableName> --item file:///tmp/<case>-item.json "${EP[@]}"
aws dynamodb get-item --table-name <TableName> --key '{"<pk>":{"S":"…"}}' "${EP[@]}"
aws dynamodb scan --table-name <TableName> "${EP[@]}"
```

## SDK note

Point the AWS SDK at the local endpoint the same way the service does in local env (custom endpoint URL). Dummy credentials satisfy the signer.

## Env alignment

| Pattern | Local value |
|---------|-------------|
| Endpoint env | `http://127.0.0.1:8000` |
| Region | any consistent value |
| Access keys | dummy `local` / `local` for Local |

## Related

- Sibling skills: `local-docker-mysql`, `local-docker-redis`, `local-docker-firestore`
