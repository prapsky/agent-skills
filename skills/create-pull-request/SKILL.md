---
name: create-pull-request
description: >-
  Create a GitHub Pull Request with a clear title, Summary, and Test plan.
  Use whenever the user asks to create a PR, open a pull request, ship a branch,
  or run gh pr create. Before opening the PR, apply relevant project skills to
  the branch diff.
---

# Create Pull Request

## Mandatory first step — apply project skills

Before creating any Pull Request:

1. List every skill directory under the project’s agent skills folder (commonly `.cursor/skills/` in the repo, or the skills bundled with this `agent-skills` repo).
2. Read each matching skill’s `SKILL.md` (and `reference.md` when linked). Prefer skills that touch the PR surface, for example:
   - `create-pull-request` (this file) — process + title/body
   - `unit-test` — when the PR adds/changes unit tests (table-driven)
   - `code-review` — optional self-check before/after open
   - `mysql-insert` — only if the PR adds/changes SQL seed scripts
   - `playwright-local-api-test` — only if the PR adds/changes Playwright API tests
   - Any other project skill that clearly applies to the diff
3. Apply those skills to the branch/diff before opening the PR.
4. Prefer the project’s documented PR template when one exists (e.g. `PULL_REQUEST_TEMPLATE.md`, contributing docs). Do not invent a conflicting format.
5. If the project has no template, use the **default body** below.

## Process

1. Confirm the user asked to create a PR (do not open one unprompted).
2. Run the skills pass above on the PR diff.
3. In parallel, inspect branch state:
   - `git status`
   - `git diff` (staged + unstaged) and `git diff <base>...HEAD`
   - `git branch -vv` / remote tracking
   - `git log` and commits unique to this branch
4. Draft title and body (project template, or default below).
5. Create a feature branch if still on the base branch.
6. Push with `-u` if needed (`git push -u origin HEAD`).
7. Create the PR with `gh pr create` and a HEREDOC body.
8. Return the PR URL. If the user asked for a review (or the project requires one), run the local `code-review` skill next — do not invent a company-specific review command.

## Required commit message (PR branches)

Whenever you commit changes that will go into a Pull Request:

- **One sentence only** — the commit subject is the full message (no body paragraphs unless the user explicitly asks).
- **Beginner-friendly** — plain language anyone on the team can understand; avoid jargon-only subjects.
- **Analogy allowed** — a short analogy is fine when it clarifies *why*.
- **Say what changed and why** — not only a ticket ID, scope prefix, or file name.

| Good | Bad |
|------|-----|
| `Stop duplicate welcome emails when staff create a contact, like not mailing the same person twice.` | `fix(notify): skip welcome on staff create` |
| `Use one phone format everywhere so lookups find the right person.` | `use normalizePhone in PutUser` |
| `Add table-driven tests for the discount calculator so edge prices stay covered.` | `Update calculator_test.go` |

Conventional-commit prefixes (`fix(scope):`, `feat:`) are fine **only when** the rest of the sentence stays plain and beginner-readable.

## Required title

- When a ticket is known: **`[TICKET] <SHORT DESCRIPTION>`** (any tracker key: `JIRA`, `LINEAR`, `GH-123`, etc.).
- When no ticket: a short plain-language title (same style as a good commit subject).
- Do **not** use a jargon-only conventional-commit title alone when a ticket exists.

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

Only if **this repo already uses** frontmatter on PRs (check recent merged PRs):

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

## Rules

- Follow the project’s existing PR style when clear from recent good PRs.
- Keep Summary as short bullets.
- Never force-push to `main`/`master`; never update git config; never skip hooks unless asked.
- Use `gh` for all GitHub PR operations.
- Never open a PR unless the user asked.

Details and examples: [reference.md](reference.md).
