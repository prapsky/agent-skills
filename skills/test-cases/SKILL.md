---
name: test-cases
description: >-
  Author positive and negative test cases for a product journey or API across
  channels (web, admin, mobile, etc.), as an HTML section or standalone file.
  Use when the user asks for test cases, QA scenarios, Given–When–Then checks,
  happy-path / negative coverage, or a test-case table for a business process.
---

# Test cases

## When to use

- User wants **test cases** for a journey, feature, or API
- Need **positive + negative** scenarios per channel (web, admin, app, partner, …)
- Turning **acceptance criteria** (often Given–When–Then) into executable checks for QA
- Adding a Test Cases section next to a `business-process` overview HTML

## When not to use

- Authoring the full **business process overview** HTML → use [`business-process`](../business-process/)
- Running automated Python API tests → use [`python-local-api-test`](../python-local-api-test/)
- Writing automated unit/integration **code** tests → use [`unit-test`](../unit-test/)

## Goals

1. Clear, channel-scoped cases a non-expert tester can follow.
2. Cover **happy path** and **failure / block** paths.
3. Tie each case to a real **API / system** (or UI-only) when known.
4. Deliver HTML and/or Markdown the team can paste into docs or a process page.

**Analogy:** Test cases are the flight checklist — not the map of the whole trip (`business-process`), and not the autopilot script (automated tests).

---

## Inputs

| Input | Required? | Notes |
|-------|-----------|-------|
| Process / feature name | Yes | What is being tested |
| Channels | Yes | Only surfaces that participate |
| Use cases + acceptance criteria | Recommended | Prefer Given–When–Then from the process/spec |
| Endpoints / systems | Recommended | From process HTML, OpenAPI, or code |
| Output format | Ask if unclear | `html-section` · `standalone-html` · `markdown` (default: ask once) |
| Output path | Yes | e.g. `{process}-test-cases.html` or embed path |

If a `business-process` HTML already exists, read its use cases, FE↔BE flows, and endpoints first — do not invent APIs.

---

## Core mapping

```text
Business process / feature
  └── Use case
        └── Acceptance criteria (Given–When–Then)
              └── Test case(s)   ← this skill
```

| Layer | Role |
|-------|------|
| Acceptance criteria | Pass/fail rules (prefer GWT) |
| **Test case** | Steps a tester performs + expected result |

One acceptance criterion may become several cases (happy path, validation error, auth fail, …).

### Given–When–Then → case fields

| GWT | Maps to |
|-----|---------|
| **Given** | Preconditions (often in Scenario or Steps) |
| **When** | Steps (what to do) |
| **Then / And** | Expected result |

---

## Workflow

```
- [ ] 1. Clarify feature, channels, output format/path
- [ ] 2. Load use cases + ACs (+ process HTML / API docs if present)
- [ ] 3. Draft case list (IDs, type, coverage gaps)
- [ ] 4. Write cases in the chosen format
- [ ] 5. Add summary strip + tester tip
- [ ] 6. Coverage checklist
- [ ] 7. Return path + counts (total / positive / negative / per channel)
```

### 1–2. Clarify and research

Ask only if missing: channels, format, where to write the file. Prefer existing ACs over inventing scenarios.

### 3. Draft with IDs

**ID convention:** `{CHANNEL}-{NN}` — zero-padded per channel.

| Channel (example) | Prefix examples |
|-------------------|-----------------|
| Website / public web | `WEB-01`, `WEB-02` |
| Admin / staff dashboard | `ADMIN-01` (or project code, e.g. `OPS-01`) |
| Mobile app | `APP-01` |
| Partner / other | Short uppercase code agreed with the user |

Use the project’s real channel codes when they already exist; otherwise pick short stable prefixes and stay consistent.

### Types

| Type | Meaning |
|------|---------|
| **Positive** | Happy path — success outcome |
| **Negative** | Blocked, validation error, ineligible, auth fail, expired, duplicate |

### Coverage expectations

- At least one **end-to-end happy path** per participating channel
- Negative cases for: missing fields, auth, duplicates, eligibility / gate blocks, expired or invalid states
- Background effects (messaging, push, jobs, onboarding) as **separate cases** when they matter
- End with a short **tester tip** (staging data, clean baseline accounts, feature flags)

---

## Table format (required columns)

| ID | Type | Scenario | Steps (what to do) | Expected result | API / system |

### Cell guidance

| Column | Write |
|--------|--------|
| **ID** | `{CHANNEL}-{NN}` |
| **Type** | `Positive` or `Negative` (use pills in HTML) |
| **Scenario** | One-line story in plain language |
| **Steps** | Numbered, concrete actions (UI clicks or API calls) |
| **Expected result** | Observable outcome (status, message, data, side effect) |
| **API / system** | Method + path, topic name, or `UI-only` |

---

## Output formats

### A. HTML section (embed in a process page)

Summary strip + table. CSS classes commonly used with process pages:

| Class | Use |
|-------|-----|
| `.tc-summary` | Counts: Total · Positive · Negative · per channel |
| `.tc-table` | Case table |
| `.pill.pos` / `.pill.neg` | Type badges |

Place as its own numbered section (e.g. after FE↔BE / impact) when merging into a `business-process` HTML — renumber surrounding sections if needed.

### B. Standalone HTML

Same table + summary + tester tip in a minimal self-contained page (reuse process-page CSS tokens if available; see [`business-process` reference](../business-process/reference.md)).

Suggested name: `{process-kebab}-test-cases.html`

### C. Markdown

```markdown
# {Process} — Test cases

**Summary:** Total N · Positive P · Negative Neg · {Channel}: n …

| ID | Type | Scenario | Steps | Expected result | API / system |
|----|------|----------|-------|-----------------|--------------|
| WEB-01 | Positive | … | 1. … | … | `POST /v1/…` |

**Tester tip:** …
```

More HTML markup examples: [reference.md](reference.md).

---

## Summary strip

Always show:

- **Total** case count  
- **Positive** / **Negative** counts  
- **Per-channel** counts  

---

## Writing rules

1. Plain language — a new QA hire should run the case without reading the code.
2. One primary outcome per case; split combined paths into separate IDs.
3. Name real endpoints and systems after a plain-language scenario line.
4. Mark async side effects clearly in **Expected result** (e.g. “message published to `{topic}`”).
5. Never put secrets, real production PII, or passwords in cases — use placeholders.

---

## Pre-publish checklist

- [ ] Every participating channel has ≥1 positive E2E case
- [ ] Negative coverage for validation, auth, and business gates that apply
- [ ] IDs unique and follow `{CHANNEL}-{NN}`
- [ ] Columns complete (no empty Expected result)
- [ ] Summary counts match the table
- [ ] Background jobs covered when they are part of the success definition
- [ ] Tester tip present
- [ ] No secrets or real credentials

---

## Pairing with other skills

| Skill | Relationship |
|-------|----------------|
| [`business-process`](../business-process/) | Journey map + ACs → feed this skill |
| [`python-local-api-test`](../python-local-api-test/) | Automate selected API cases with Python after cases exist |
| [`unit-test`](../unit-test/) | Automated in-process unit tests (table-driven code), not QA checklists |
| [`local-docker-mysql`](../local-docker-mysql/) | Seed data for cases that need DB fixtures |
| [`test-data-conventions`](../test-data-conventions/) | Synthetic phones (`+6285YYMMDDxxx`) and UUID string IDs |

---

## Additional resources

- HTML summary/table snippets and type pills: [reference.md](reference.md)
