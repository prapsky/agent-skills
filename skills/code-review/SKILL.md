---
name: code-review
description: >-
  Reviews a PR or branch against agreed docs (spec, plan, requirements,
  acceptance criteria) and returns a clear Yes/No merge verdict — optionally
  posted on GitHub as APPROVE or REQUEST_CHANGES.
  Trigger on: review this PR, code review, review before merge, check this
  branch, formal review, review and approve, is this PR ready, review against
  the spec, give a verdict, post a review on GitHub, request changes, should
  we merge this, look at this PR, audit this change, check the diff.
  Not for fixing review comments → use answer-code-reviews instead.
  Not for writing or rewriting the feature → implement first, then review.
  Not for a vague "look at this file" with no PR, branch, or goal → ask for
  scope first.
---

# Code review

## What this skill does

- Reviews a **code change** (PR or branch) against what was **agreed** (spec,
  plan, requirements, acceptance criteria).
- Returns a clear verdict: **Yes** (merge) or **No** (blocked — see Must fix).
- Posts the review on GitHub when asked.

**Analogy:** Building inspection — the docs are the blueprint; the PR is the
finished build.

---

## Keep it short (required)

Use **4 sections only**. Plain English. One line per item.

| Section | Rule |
|---------|------|
| **Can we merge?** | **Yes** or **No** only — must match Must fix (see below) |
| **Must fix** | Blockers only. Write `None` if empty. |
| **Optional** | Nice-to-haves — not required before merge. Write `None` if empty. |
| **After merge** | Deploy + smoke test ops steps, not code fixes. |

### Verdict consistency (hard rules — do not break)

| **Can we merge?** | **Must fix** |
|-------------------|--------------|
| **Yes** | `None` |
| **No** | One or more items listed |

1. Must fix is `None` → Can we merge? is **Yes** — never "after fixes", never
   "with notes".
2. Must fix has items → Can we merge? is **No**.
3. Optional items never change the verdict. They are follow-up ideas, not
   blockers.
4. After merge is deploy/smoke/ops — not code changes. Put nothing there that
   blocks the merge.
5. Inline GitHub comments → **Must fix only**. Put Optional items in the review
   body; no line comments for Optional.
6. `post: true` → set the GitHub review `event` from the verdict (see
   [GitHub approve checkbox](#github-approve-checkbox-required)) — writing
   "Yes" in the body alone is not enough.

**Forbidden phrases:** "Yes, after fixes", "Should fix", "Minor",
"Merge readiness", long intro paragraphs, "What looks good" (unless asked).

### Chat output (default)

```markdown
**Can we merge?** Yes | No

| | |
|---|---|
| **Must fix** | … or None |
| **Optional** | (1) … or None — not blocking |
| **After merge** | Deploy …; smoke … |
```

| Audience | Rule |
|----------|------|
| **Chat** | Compact 4-row table only. No full review unless asked. |
| **GitHub body** | Same 4 sections. ≤25 lines. One line per bullet. |
| **Inline comments** | Must fix only — label + 1–2 sentences + fix. |

---

## What you need

| Parameter | Required? | Meaning |
|-----------|-----------|---------|
| **`repo`** | Yes (GitHub) | `owner/repo` |
| **`pr`** | PR **or** branch | PR number or URL |
| **`branch`** | PR **or** branch | Branch name if no PR yet |
| **`docs`** | Recommended | Paths or links to spec / plan / requirements / ACs |
| **`post`** | No | `true` = post on GitHub · `false` = report only (default) |

When `docs` is missing, discover them from the PR body, linked issues, `docs/`,
`README`, tickets — or ask once.

---

## Docs to read first

| Doc type | Examples | Why |
|----------|----------|-----|
| Spec / contract | OpenAPI, ADR, API contract, design doc | Agreed shapes and behavior |
| Plan | Phase plan, implementation notes, ticket description | What must be built |
| Requirements | PRD, REQUIREMENTS.md, user stories | Product rules |
| Acceptance criteria | Given–When–Then, checklist in ticket | Pass/fail for the workflow |
| Test plan | VALIDATION, CI notes, test list | What should pass |
| Conventions | CONTRIBUTING, style guide, TESTING.md | Team norms |

Warn the requester when a doc is still marked **DRAFT** — do not treat it as
final.

---

## Related user workflows

Check that the **user workflow** still works — not only that APIs or types match.

| Source | What you get |
|--------|--------------|
| Acceptance criteria / user stories | Happy path + edges |
| Requirements / PRD | Actor rules, out-of-scope |
| Plan / ticket | Must-haves vs nice-to-haves |

Cite the story / AC / REQ ID in findings when available.

---

## Steps

### 1. Find the change

```bash
gh pr view <N> --repo <owner/repo> \
  --json title,body,headRefName,baseRefName,headRefOid,files,additions,deletions,state,url
gh pr diff <N> --repo <owner/repo>
```

Branch only:

```bash
git fetch origin
git log origin/<base>..origin/<branch> --oneline
git diff origin/<base>...origin/<branch>
```

### 2. Open the local code

```bash
git fetch origin <headRefName> && git checkout <headRefName>
```

Do not stash dirty work without asking.

### 3. Read agreement docs

At minimum: stated goal + any spec/plan/requirements/ACs linked for this change.

### 4. Review code — four passes

| Pass | Ask |
|------|-----|
| **A** | Match the agreement? (API, fields, DB, status codes, flags, UX contract) |
| **B** | Match plan + user workflow? (must-haves, stories, ACs) |
| **C** | Tests / CI good? (run relevant tests; `gh pr checks` when available) |
| **D** | Safe + clean? (secrets, authz, error handling, team conventions) |

### 5. Label each finding

| Label | Section | When |
|-------|---------|------|
| **Must fix** | Must fix | Breaks agreement, security, data safety, or required tests |
| **Optional** | Optional | Real gap, weak test, style, docs, or small tidy |
| **After merge** | After merge | Deploy steps, smoke tests, ops notes |

### 5b. Merge decision

Set **Can we merge?** from **Must fix** only:

| **Must fix** | **Can we merge?** |
|--------------|-------------------|
| `None` | **Yes** |
| Has items | **No** |

Do not use "Yes, after fixes" or "Approved with notes".

### GitHub approve checkbox (required)

When **`post: true`**, the GitHub review `event` must match:

| **Can we merge?** | GitHub `event` | Reviewer UI |
|-------------------|----------------|-------------|
| **Yes** | `APPROVE` | Green check — counts toward required approvals |
| **No** | `REQUEST_CHANGES` | Changes requested — blocks merge |

**Never** use `COMMENT` when `post: true` — that only adds text and does not
check Approve, even if the body says "Can we merge? Yes".

Use `COMMENT` only when the user explicitly asks for feedback without
approving or blocking.

### 6. Share the result

- **`post: false`** → chat compact summary only.
- **`post: true`** → post short GitHub body with the correct `event`; inline
  comments for Must fix only; give the user the compact chat summary.

Shortcut when **Can we merge?** is **Yes**:

```bash
gh pr review <N> --repo <owner/repo> --approve --body "$(cat <<'EOF'
## PR review — …
**Can we merge?** Yes

### Must fix
None

### Optional
_Not required before merge._
None

### After merge
- Deploy: …
- Smoke: …
EOF
)"
```

---

## GitHub review body template

```markdown
## PR review — <ticket or feature> @ `<short-sha>`

**Can we merge?** Yes | No

### Must fix
- <one line each> | None

### Optional
_Not required before merge._
- <one line each> | None

### After merge
- Deploy: <services or targets>
- Smoke: <one-line check>
```

Before posting, verify:

- Must fix = None ↔ Can we merge? = Yes ↔ `event: APPROVE`
- Must fix has items ↔ Can we merge? = No ↔ `event: REQUEST_CHANGES`

Hard limits:

- **≤25 lines** for the posted body
- One bullet = one finding
- No inline comments for Optional items

---

## Cross-repo / dependency notes

When the change depends on another PR or migration:

1. Call out the dependency in After merge or Must fix (Must fix only when this
   PR is unsafe to merge alone).
2. Prefer a sensible order: schema/migrations → backend → clients, unless the
   team's docs say otherwise.

---

## References

`gh api` posting examples, doc-discovery path patterns, and pass-checklist
details: [references/reference.md](references/reference.md).

**Pair skill:** [`answer-code-reviews`](../answer-code-reviews/) handles the
author's response to review findings.
