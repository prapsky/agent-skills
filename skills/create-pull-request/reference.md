# Create Pull Request — reference

## Good vs weak PR titles

| Good | Weak |
|------|------|
| `[ABC-123] Return club name on lead detail so the UI can show location` | `fix(leads): add club` |
| `Document local Docker MySQL and Redis for API tests` | `Update skills` |
| `[GH-42] Fix empty discount when cart has mixed tax` | `misc fixes` |

## Default body example

```markdown
## Summary
- Point local API tests at Docker MySQL and Redis instead of remote staging.
- Keep seed + Playwright skills aligned so agents do not invent a second path.

## Test plan
- [ ] Read both skill files and confirm staging DB/Redis are forbidden
- [ ] Dry-run the documented `docker start` / `docker run` commands on a machine with Docker
```

## Optional frontmatter (only if the repo already uses it)

**Service / function list** (some backend monorepos):

```markdown
---
functions:
  - <deployable-function-or-service-name>
---
```

**Gateway / env** (some API gateway repos):

```markdown
---
environment: staging
type: web
---
```

Infer keys from recent merged PRs in that repo. If unsure, ask — do not copy another company’s template.

## After `gh pr create`

1. Paste the PR URL in chat.
2. If the user wants a review, follow `skills/code-review/` (or the project’s review skill).
3. Do not require a company-specific slash command.

## Safety

- No secrets in the PR body.
- No production deploys from this skill.
- No force-push to default branches.
