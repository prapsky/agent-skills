---
name: test-data-conventions
description: >-
  Use this skill whenever you need to create a phone number or a string ID for
  test seeds, fixtures, local API request bodies, or QA examples. Triggers on:
  "create test phone", "generate seed phone", "+628…", "new UUID for test",
  "string ID for fixture", "test data phone format", or any moment the agent is
  about to type a phone number or primary-key string ID into a seed script,
  INSERT statement, or request body. Apply before writing any seed, not after.
  Never invent random 62812… phones or timestamp-only IDs — always use the
  +6285YYMMDDxxx pattern from today's date.
---

# Test data conventions (phones + string IDs)

Apply these rules **before** you write any phone number or string ID into a seed
script, SQL statement, request body, or test fixture. Using a predictable format
makes seeds traceable, avoids collisions, and simplifies cleanup.

## When to use

| Use this skill | Skip |
|---------------|------|
| Creating a new phone for any seed or fixture | Reproducing a frozen bug with an exact user-supplied phone |
| Creating a new string primary key | FK / lookup columns — discover the real row instead |
| Writing request bodies that include phones or IDs | User supplies the exact value explicitly |

## Phone numbers

**Format:** `+6285YYMMDDxxx`

| Part | Meaning |
|------|---------|
| `+6285` | Fixed prefix |
| `YY` | Current year, 2 digits |
| `MM` | Current month, 2 digits (`01`–`12`) |
| `DD` | Current day, 2 digits (`01`–`31`) |
| `xxx` | Sequence in this seed/run, starting at `001`, then `002`, … |

**Example — today is 15 September 2026:**
- First phone: `+6285260915001`
- Second phone: `+6285260915002`

### Store variants (same digits, different prefix)

| Context | Value |
|---------|-------|
| Canonical / E.164 with plus | `+6285260915001` |
| Stores that keep digits only (no `+`) | `6285260915001` |

Do **not** invent random `62812…` phones, timestamp-only numbers, or sequential integers.

### Quick generator

```python
from datetime import date

def test_phone(seq: int = 1, *, with_plus: bool = True) -> str:
    """seq starts at 1 → xxx = 001, 002, …"""
    body = f"6285{date.today().strftime('%y%m%d')}{seq:03d}"
    return f"+{body}" if with_plus else body

# test_phone(1)  → "+6285260915001"
# test_phone(2, with_plus=False)  → "6285260915002"
```

## String IDs (primary keys)

When you are creating a **string primary key** for a new row/document/item:

1. Use a **valid UUID v4** (RFC 4122) — `str(uuid.uuid4())`.
2. Do not use slug IDs like `case-seed-1` or `ticket-rec-abc123` unless the schema or API **requires** a non-UUID format. If so, note why.

```python
import uuid

def new_string_id() -> str:
    return str(uuid.uuid4())

# e.g. "a3f1c2e4-9b8d-4e2a-91f0-7c6d5b4a3210"
```

## Foreign keys are different — always discover

For **FK / lookup columns**: find the real row with a `SELECT … WHERE <unique_col> = '…'` query. Do not invent UUID foreign keys that do not exist in the database — they will cause FK constraint failures.

```sql
-- Correct: discover the real parent id
SELECT id FROM <fk_table> WHERE <unique_name_col> = '<known-fixture-name>' LIMIT 1;
```

## Checklist

```
- [ ] New phones use +6285YYMMDDxxx (or 6285… without + when the store requires it)
- [ ] xxx increments per phone in the same case (001, 002, …)
- [ ] YYMMDD is today's date (unless reproducing a frozen bug date the user gave)
- [ ] New string IDs are valid UUIDs
- [ ] FK ids come from discovery SELECTs, not invented UUIDs
```

## Related skills

| Skill | When |
|-------|------|
| `local-docker-mysql` / `redis` / `dynamodb` / `firestore` | Seeding stores with phones/IDs |
| `python-local-api-test` | Request bodies that create phones/IDs |
| `playwright-local-api-test` | UI flows that type phones/IDs into forms |

## References

Read `references/reference.md` for: multi-phone batch generator, UUID batch generator, more format examples, and a worked INSERT example with both phone and UUID.
