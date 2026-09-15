---
name: create-pull-request
description: >-
  Opens a GitHub Pull Request with a well-structured title, summary, and test
  plan — applying matching project skills to the diff first.
  Trigger on: create a PR, open a pull request, ship this branch, gh pr create,
  push and open, submit for review, make a draft PR, raise a PR, open a draft,
  PR this up, put up a PR, file a pull request, create draft pull request.
  Not for merging an open PR, and not for reviewing one — use the code-review
  skill for a merge verdict.
---

# Create Pull Request

## Apply project skills first

Before opening any PR, scan the project's agent skills folder (commonly
`.cursor/skills/` or the `agent-skills` repo). Read each matching skill's
`SKILL.md` (and its `references/reference.md` when linked). Apply every skill
that touches the PR surface:

| Skill | Apply when |
|-------|-----------|
| `unit-test` | PR adds or changes unit tests |
| `code-review` | Optional self-check before/after opening |
| `local-docker-mysql` / `-redis` / `-dynamodb` / `-firestore` / `-pubsub` / `-firebase-remote-config` | PR adds or changes those Docker seed skills/recipes |
| `python-local-api-test` | PR adds or changes local Python API probe scripts |
| `playwright-local-api-test` | PR adds or changes Playwright UI tests (not HTTP API) |
| `test-data-conventions` | Seeds or fixtures invent phones or string IDs |

Prefer the project's existing PR template (`PULL_REQUEST_TEMPLATE.md`,
contributing docs) over the default below. Do not invent a conflicting format.

---

## Process

1. Confirm the user asked to create a PR. Never open one unprompted.
2. Run the skills pass above on the diff.
3. Inspect the branch in parallel:
   ```bash
   git status
   git diff && git diff <base>...HEAD
   git branch -vv
   git log --oneline <base>..HEAD
   ```
4. Draft the title and body (project template, or default below).
5. Create a feature branch if still on the base branch.
6. Push with `-u` if needed: `git push -u origin HEAD`
7. Create the PR:
   ```bash
   gh pr create --draft --title "<title>" --body "$(cat <<'EOF'
   <body>
   EOF
   )"
   ```
8. Return the PR URL. When the user wants a review, run the `code-review` skill
   next — do not invent a company-specific review command.

---

## Required commit message (PR branches)

One sentence only. Plain language anyone can understand.

| Good | Bad |
|------|-----|
| `Stop duplicate welcome emails when staff create a contact, like not mailing the same person twice.` | `fix(notify): skip welcome on staff create` |
| `Use one phone format everywhere so lookups find the right person.` | `use normalizePhone in PutUser` |
| `Add table-driven tests for the discount calculator so edge prices stay covered.` | `Update calculator_test.go` |

- Say **what changed and why** — not only a ticket ID, scope prefix, or file name.
- A short analogy is fine when it clarifies *why*.
- Conventional-commit prefixes (`fix(scope):`) are acceptable **only when** the
  rest of the sentence stays plain and beginner-readable.

---

## Required title

- Ticket known → **`[TICKET] <SHORT DESCRIPTION>`** (any key: `JIRA`, `LINEAR`, `GH-123`…).
- No ticket → a short plain-language title in the same style as a good commit subject.
- Do not use a jargon-only conventional-commit title when a ticket exists.

---

## Default body shape

Use when the project has no stronger template:

```markdown
## Summary
- <what changed>
- <why / key behavior>
- <out-of-scope notes if needed>

## Test plan
- [ ] <commands or checks>
```

### Optional YAML frontmatter

Add **only** when this repo already uses frontmatter on PRs (check recent merged PRs):

```markdown
---
# project-specific keys, e.g. service name, environment, type
---

## Summary
- …

## Test plan
- [ ] …
```

Do not invent frontmatter keys the repo does not already use.

---

## Rules

- Follow the project's existing PR style when clear from recent good PRs.
- Keep Summary as short bullets.
- Never force-push to `main`/`master`, never update git config, never skip hooks
  unless explicitly asked.
- Use `gh` for all GitHub PR operations.
- Never open a PR unless the user asked.

---

## References

Title/body examples, optional frontmatter patterns, and post-create steps:
[references/reference.md](references/reference.md).
