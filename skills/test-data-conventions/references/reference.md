# Test data conventions — reference

## Phone generator (single + batch)

```python
from datetime import date

def test_phone(seq: int = 1, *, with_plus: bool = True) -> str:
    """
    Generate a deterministic test phone from today's date.

    Format: +6285YYMMDDxxx
    seq starts at 1 → xxx = 001, 002, …
    """
    body = f"6285{date.today().strftime('%y%m%d')}{seq:03d}"
    return f"+{body}" if with_plus else body


def test_phones(count: int, *, with_plus: bool = True) -> list[str]:
    """Return `count` phones in one call."""
    return [test_phone(i, with_plus=with_plus) for i in range(1, count + 1)]


# Examples (date = 2026-09-15):
# test_phone(1)                   → "+6285260915001"
# test_phone(2, with_plus=False)  → "6285260915002"
# test_phones(3)                  → ["+6285260915001", "+6285260915002", "+6285260915003"]
```

## UUID generator (single + batch)

```python
import uuid

def new_string_id() -> str:
    """Valid UUID v4 for new string primary keys."""
    return str(uuid.uuid4())


def new_string_ids(count: int) -> list[str]:
    """Return `count` UUIDs in one call."""
    return [new_string_id() for _ in range(count)]


# Examples:
# new_string_id()   → "a3f1c2e4-9b8d-4e2a-91f0-7c6d5b4a3210"
# new_string_ids(2) → ["<uuid1>", "<uuid2>"]
```

## Worked INSERT example

```sql
-- Seeding one row with a UUID PK and a date-based phone
-- (today = 15 Sep 2026 → phone 001 = +6285260915001)
INSERT INTO leads (id, phone, parent_id, remarks)
SELECT
  'a3f1c2e4-9b8d-4e2a-91f0-7c6d5b4a3210',   -- new UUID PK
  '+6285260915001',                            -- test_phone(1)
  p.id,                                        -- FK discovered, not invented
  'seed:acq-2937'
FROM parents p
WHERE p.name = 'known-fixture-name'
LIMIT 1;
```

## Format matrix

| Field type | Local policy | Example (2026-09-15, seq 1) |
|------------|-------------|------------------------------|
| Phone (E.164) | `+6285YYMMDDxxx` | `+6285260915001` |
| Phone (digits only) | `6285YYMMDDxxx` | `6285260915001` |
| String PK | Valid UUID v4 | `a3f1c2e4-9b8d-4e2a-91f0-7c6d5b4a3210` |
| FK / lookup id | Discover with SELECT | Do not invent |
| Bug-replication phone | Exact user-supplied value | Preserve as-is |

## Seed tag convention

When the schema has a `remarks`, `tag`, `notes`, or equivalent free-text column, set the value to `seed:<ticket>` (e.g. `seed:acq-2937`). This makes ticket-scoped cleanup safe:

```sql
DELETE FROM <table> WHERE remarks LIKE 'seed:acq-2937%';
```

## Handoff JSON shape

Always write seed details to `/tmp/<case>-seed.json` — no secrets:

```json
{
  "case": "acq-2937",
  "date": "2026-09-15",
  "stores_seeded": ["mysql"],
  "phones": ["+6285260915001"],
  "ids": ["a3f1c2e4-9b8d-4e2a-91f0-7c6d5b4a3210"],
  "cleanup_tag": "seed:acq-2937%"
}
```

## Related

- `local-docker-mysql` — INSERT patterns that use phones and UUIDs
- `python-local-api-test` — request bodies that contain phones and IDs
- `playwright-local-api-test` — UI form inputs with phones and IDs
