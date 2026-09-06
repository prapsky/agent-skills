# answer-code-reviews — reference

Companion to [SKILL.md](SKILL.md).

## Pair with `code-review`

| Skill | Role |
|-------|------|
| [`code-review`](../code-review/) | Reviewer posts Yes/No + Must fix / Optional / After merge |
| `answer-code-reviews` | Author fixes, replies, optionally resolves threads |

Typical flow:

1. Reviewer runs `code-review` with `post=true`.
2. Author runs `answer-code-reviews` with `commit push reply` (and `resolve` if desired).

## Doc discovery (when PR links planning)

| Clue in PR | Where to look |
|------------|---------------|
| Link to issue / ticket | Description, ACs, attachments |
| `See docs/...` or `See .planning/...` | Follow the path in that workspace |
| Spec filename in review body | Repo `docs/`, `specs/`, OpenAPI |
| No links | Ask once for the agreement doc, or use PR description + diff only |

## Classifying mixed review styles

Reviewers are inconsistent. Normalize before you work:

| You see | Action bucket |
|---------|---------------|
| `Can we merge? No` + Must fix list | Fix all Must fix before asking for re-review |
| `REQUEST_CHANGES` | Same as Must fix |
| `APPROVE` + Optional notes | Optional only — fix if cheap or asked |
| `COMMENT` with mixed severity | Parse labels; ask if unclear |

## Focused test examples (pick what fits the repo)

```bash
# Go
go test ./path/to/package/... -run 'RelevantTest' -count=1 -v

# Node / JS
npm test -- --grep 'relevant case'
# or: pnpm test -- relevant.test.ts

# Python
pytest path/to/test_file.py -k 'relevant'

# Make / task runners
make test-unit PKG=./internal/api
```

Prefer the smallest command that covers the changed behavior.

## Reply hygiene

- One finding → one reply (or one numbered section in a body-only comment)
- Always include `<short-sha>` when you fixed something
- Do not resolve a thread before the fix is pushed (unless the finding was acknowledgment-only and the user asked)
