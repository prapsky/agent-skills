---
name: mysql-insert
description: >-
  Generate safe, DBeaver-ready MySQL INSERT/seed SQL. Use when the user asks
  for MySQL insert queries, seed data, staging/local fixtures, or DBeaver SQL.
---

# MySQL insert

## When to use

- User wants MySQL `INSERT` / seed SQL
- User mentions DBeaver and needs copy-paste insert statements

## When not to use

- Production writes unless the user **explicitly** asks for production
- Running inserts with `.env` credentials unless the user asks to execute
- Firestore / Dynamo / API setup (out of scope — MySQL only)
- During PR creation: **skip** unless the PR adds/changes SQL seed scripts

## Goals

1. Copy-paste SQL that works in **DBeaver** (prefer one statement).
2. Resolve foreign keys with `JOIN` / subqueries — do not invent IDs.
3. Include discovery → insert → verify → cleanup.

## Workflow

```
- [ ] 1. Clarify table, columns, and values
- [ ] 2. Discovery SELECT (FK lookups non-NULL)
- [ ] 3. INSERT (prefer INSERT … SELECT when FKs needed)
- [ ] 4. Verify SELECT
- [ ] 5. Cleanup DELETE
```

### 1. Clarify

Ask only if missing: target DB/table, required columns, how to find FK rows (e.g. by name).

### 2. Discovery first

```sql
SELECT id, <label_column>
FROM <parent_table>
WHERE <label_column> = '<known_value>'
LIMIT 5;
```

### 3. Prefer `INSERT … SELECT` when FKs exist

Avoid multi-statement `SET @var = …` as the default (DBeaver often errors with `1064` near `SET`).

```sql
INSERT INTO <child_table> (id, parent_id, name, created_at)
SELECT
  '<new-id>',
  p.id,
  'Seed Name',
  UNIX_TIMESTAMP()
FROM <parent_table> p
WHERE p.<label_column> = '<known_value>'
LIMIT 1;
```

For tables with no FKs, a plain `INSERT INTO … VALUES (…)` is fine.

### 4. Verify + cleanup

```sql
SELECT * FROM <table> WHERE id = '<new-id>';

DELETE FROM <table> WHERE id = '<new-id>';
```

## Security

- Staging/local by default; warn before production SQL
- Never paste DB passwords or commit secrets
- Do not auto-run remote inserts unless the user asks

## Output shape

1. One-line what is being seeded  
2. Discovery SQL  
3. Insert SQL  
4. Verify SQL  
5. Cleanup SQL  

Keep it short. Use the user’s real table/column names when given.

## More patterns

See [reference.md](reference.md).
