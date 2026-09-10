---
name: test-data-conventions
description: >-
  Rules for synthetic phones and string IDs when seeding Docker stores, writing
  local API request bodies, UI test fixtures, or QA examples. Use whenever the
  agent creates a phone number or a string ID for test/seed data.
---

# Test data conventions (phones + string IDs)

Apply these rules **whenever you create** a phone number or a string ID for seeds, fixtures, local API bodies, or test-case examples — unless the user supplies an exact value to reproduce.

## Phone numbers

**Format:** `+6285YYMMDDxxx`

| Part | Meaning |
|------|---------|
| `+6285` | Fixed prefix |
| `YY` | Current year, 2 digits |
| `MM` | Current month, 2 digits (`01`–`12`) |
| `DD` | Current day, 2 digits (`01`–`31`) |
| `xxx` | Sequence in this seed/run, starting at `001`, then `002`, … |

**Example:** on **11 September 2026**, the first phone is `+6285260911001`, the second is `+6285260911002`.

### Store / API variants

Keep the same digits; only the leading `+` may change to match the field:

| Context | Value |
|---------|--------|
| Canonical / E.164 with plus | `+6285260911001` |
| Stores that keep digits only (no `+`) | `6285260911001` |

Do **not** invent random `62812…` or timestamp-based phones when creating new test data.

### Quick generator (Python)

```python
from datetime import date

def test_phone(seq: int = 1, *, with_plus: bool = True) -> str:
    """seq starts at 1 → xxx = 001, 002, …"""
    body = f"6285{date.today().strftime('%y%m%d')}{seq:03d}"
    return f"+{body}" if with_plus else body
```

## String IDs

When a field is a **string ID** you are creating (primary key, entity id, document id, request `id`, etc.):

1. Use a **valid UUID** (RFC 4122), e.g. `str(uuid.uuid4())` → `a3f1c2e4-9b8d-4e2a-91f0-7c6d5b4a3210`.
2. Do **not** use slug ids like `case-seed-1` or `ticket-rec-abc123` unless the schema/API **requires** a non-UUID format (then follow that contract and note why).

### Foreign keys are different

For **FK / lookup** columns: **discover** the real row by a unique name (`SELECT … WHERE <unique_col> = …`). Do **not** invent UUID foreign keys that do not exist.

```python
import uuid

def new_string_id() -> str:
    return str(uuid.uuid4())
```

## Checklist

```
- [ ] New phones use +6285YYMMDDxxx (or 6285… without + when the store requires it)
- [ ] xxx increments per phone in the same case (001, 002, …)
- [ ] YYMMDD is today's date (unless reproducing a frozen bug date the user gave)
- [ ] New string IDs are valid UUIDs
- [ ] FK ids still come from discovery SELECTs, not invented UUIDs
```

## Related skills

| Skill | When |
|-------|------|
| `local-docker-mysql` / `redis` / `dynamodb` / `firestore` | Seeding stores |
| `python-local-api-test` | Request bodies that create phones/ids |
| `playwright-local-api-test` | UI flows that type phones/ids |
| `test-cases` / `unit-test` | Examples and fixtures with phones/string ids |
