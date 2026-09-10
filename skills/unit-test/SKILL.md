---
name: unit-test
description: >-
  Write or rewrite automated unit tests, preferring table-driven cases that
  match the package’s existing style. Use when the user asks for unit tests,
  table-driven tests, to convert nested t.Run essays into a case table, or to
  add coverage for a pure function / mapper / parser.
---

# Unit test

## When to use

- User asks to **write unit tests**, **add coverage**, or **make table-driven tests**
- Converting scattered `t.Run` / separate `TestFoo_Case` functions into one case table
- Testing pure helpers: mappers, parsers, normalizers, resolvers, validators

## When not to use

- QA journey checklists / Given–When–Then docs → [`test-cases`](../test-cases/)
- Local Python API checks with seed data → [`python-local-api-test`](../python-local-api-test/)
- Changing production behavior unless the user also asked for a code fix

**Analogy:** Unit tests are a **checklist of inputs → expected outputs**. Prefer one table you can scan in 30 seconds over many separate essays.

---

## Goals

1. Match the **package’s existing test style** first (naming, asserts, helpers).
2. Prefer **table-driven** tests when there are 2+ related scenarios.
3. Cover **happy path + edges** that the code actually branches on.
4. Run the **focused** test command and report pass/fail.

---

## Inputs

| Input | Required? | Notes |
|-------|-----------|-------|
| Function / file under test | Yes | Path or symbol |
| Behavior to lock in | Yes | Spec, PR, or current code |
| Language / test framework | Infer | From nearby `*_test.go`, `*.test.ts`, etc. |
| Existing sibling tests | Recommended | Copy style from the same package |

Ask only if the target function or expected behavior is unclear.

---

## Workflow

```
- [ ] 1. Read the code under test + 1–2 sibling tests in the same package
- [ ] 2. List cases (happy path, invalid, empty, case/trim, defaults)
- [ ] 3. Write or rewrite as table-driven (or package-native equivalent)
- [ ] 4. Keep assertions one clear expect per case
- [ ] 5. Run focused tests; fix failures
- [ ] 6. Summarize what cases were added (short table is fine)
```

### 1. Match the neighborhood

Before inventing a style:

1. Open nearby `*_test.go` / test files in the **same package**.
2. Reuse: assert library (`testify`, stdlib), `tt` vs `tc`, `t.Parallel()`, helpers.
3. Only introduce a new pattern when the package has none.

### 2. Default shape (Go)

```go
func TestThing(t *testing.T) {
	tests := []struct {
		name    string
		input   string
		want    string
		wantErr string // optional: non-empty means expect error containing this
	}{
		{name: "happy path", input: "a", want: "A"},
		{name: "invalid", input: "{", wantErr: "parse"},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got, err := Thing(tt.input)
			if tt.wantErr != "" {
				require.Error(t, err)
				assert.Contains(t, err.Error(), tt.wantErr)
				return
			}
			require.NoError(t, err)
			assert.Equal(t, tt.want, got)
		})
	}
}
```

**Rules:**

| Do | Don’t |
|----|--------|
| One row per scenario; `name` is human-readable | Duplicate almost-identical `t.Run` blocks with no shared table |
| Fields: inputs + `want` (+ `wantErr` when needed) | Giant setup shared across unrelated behaviors in one table |
| `require` for must-stop checks; `assert` for value checks | Ignore errors and only assert the happy return |
| Keep production code unchanged when only tests were requested | “Improve” production logic while rewriting tests |

Other languages: same idea — **cases array + loop + named subtest** (Jest `it.each`, pytest parametrize, etc.). Prefer whatever the repo already uses.

### 3. Case coverage checklist

Include a row when the code has a real branch for it:

| Kind | Examples |
|------|----------|
| Happy path | Known key → mapped value |
| Invalid input | Bad JSON, malformed payload |
| Empty / whitespace | `""`, `"   "` |
| Case / trim | `"Instagram"`, `" INSTAGRAM "` |
| Alias / literal | Legacy `"organic"` string |
| Default / fallback | Unknown key → Paid; no tracker → Organic |
| Embedded / fixture load | Smoke that real asset keys still resolve |

Do **not** invent branches the code does not have.

### 4. Rewrite existing tests

When asked to “make it table-driven”:

1. Preserve **all** existing scenarios (do not drop coverage).
2. Merge related functions (`TestFoo_Valid` + `TestFoo_Invalid` → `TestFoo` with `wantErr`).
3. Split a multi-assert `t.Run` into **one row per input** when inputs differ.
4. Leave unrelated smoke tests alone if a table does not help (e.g. one-shot init checks can stay, or use a small key→value table).

### 5. Run tests

- Prefer a **narrow** `-run` / filter matching the new tests.
- Use the project’s usual flags (e.g. Go: `go test -mod=mod ./path -run 'Pattern' -count=1`).
- If deps download is slow, wait; do not claim pass without a green result.

---

## Pairing with other skills

| Skill | Relationship |
|-------|----------------|
| [`test-cases`](../test-cases/) | Human QA checklist → may inspire unit cases; not a substitute |
| [`python-local-api-test`](../python-local-api-test/) | HTTP-level local Python check; unit tests stay in-process |
| [`code-review`](../code-review/) | Reviewers often ask for table-driven coverage — use this to add it |
| [`answer-code-reviews`](../answer-code-reviews/) | If review says “make table-driven”, fix with this skill then reply |

---

## Pre-merge checklist

- [ ] Style matches sibling tests in the package
- [ ] Table covers every previously tested scenario (on rewrites)
- [ ] Edge cases for real branches are present
- [ ] Focused test command passed
- [ ] No production behavior change unless requested

---

## Additional resources

- Before/after Go examples and anti-patterns: [reference.md](reference.md)
