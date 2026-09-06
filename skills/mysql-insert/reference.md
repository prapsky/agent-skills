# MySQL insert — patterns

## DBeaver

- Prefer **Execute SQL Statement** (Ctrl+Enter) — one statement at a time
- Avoid `SET @var` multi-scripts unless script mode is confirmed
- Avoid trailing `--` comments on the same line as values if the client mis-parses

## `INSERT … SELECT` (FK by name)

```sql
INSERT INTO child (id, parent_id, name)
SELECT 'seed-1', p.id, 'Example'
FROM parent p
WHERE p.name = 'Known Parent'
LIMIT 1;
```

## Plain `INSERT`

```sql
INSERT INTO items (id, name, created_at)
VALUES ('seed-1', 'Example', UNIX_TIMESTAMP());
```

## Unix timestamp → GMT+7 (read-only check)

```sql
SELECT
  id,
  created_at,
  CONVERT_TZ(FROM_UNIXTIME(created_at), '+00:00', '+07:00') AS created_at_gmt7
FROM <table>
WHERE id = '<id>';
```

## Cleanup

```sql
DELETE FROM <table> WHERE id = '<id>';
-- or a clear seed marker:
-- DELETE FROM <table> WHERE remarks LIKE 'seed:%';
```
