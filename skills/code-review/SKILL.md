---
name: code-review
description: >-
  Lead a pull request or branch code review against agreed docs (spec, plan,
  requirements, acceptance criteria), run key checks, and optionally post a
  GitHub review with a clear Yes/No merge verdict. Use when reviewing a PR,
  reviewing a branch before open, or when the user asks for a formal code review.
---

# Code review

## What this skill does

- Reviews a **code change** (PR or branch) against what was **agreed** (spec, plan, requirements, acceptance criteria)
- Gives a clear verdict: **Yes** (merge) or **No** (blocked — see Must fix)
- Can post the review on GitHub when asked

**Analogy:** Building inspection — the docs are the blueprint; the PR is the finished build.

**Not for:**
- Writing or rewriting the feature (implement first, then review)
- Replying as the author fixing review comments (use your team’s reply/fix skill if you have one)
- Vague “look at this file” with no PR/branch and no goal — ask for a PR, branch, or stated goal first

---

## Keep it short (required)

Use **4 sections only**. Plain English. One line per item.

| Section | Rule |
|---------|------|
| **Can we merge?** | **Yes** or **No** only — must match **Must fix** (see below) |
| **Must fix** | Blockers only. Write `None` if empty. |
| **Optional** | Nice-to-haves — **not required before merge**. Write `None` if empty. |
| **After merge** | Deploy + smoke test (ops steps, not code fixes) |

### Verdict must match Must fix (required)

| **Can we merge?** | **Must fix** |
|-------------------|--------------|
| **Yes** | `None` |
| **No** | One or more items listed |

**Consistency rules (do not break these):**

1. If **Must fix** is `None` → **Can we merge?** MUST be **Yes** (never "after fixes", never "with notes").
2. If **Must fix** has items → **Can we merge?** MUST be **No**.
3. **Optional** items do **not** change the verdict. They are follow-up ideas, not merge blockers.
4. **After merge** is deploy/smoke/ops — not code changes. Do not put code fixes there.
5. **Inline GitHub comments** → **Must fix only**. Put Optional items in the review body only (no line comments).
6. **`post: true`** → set the GitHub review **event** from the verdict (see [GitHub approve checkbox](#github-approve-checkbox-required)) — **Yes** in the body alone is not enough.

**Do not use:** What looks good, Should fix, Minor, Merge readiness, "Yes, after fixes", or long intro paragraphs.

| Audience | Rule |
|----------|------|
| **Chat** | Table only (4 rows). No full review unless asked. |
| **GitHub body** | Same 4 sections. ≤25 lines. One line per bullet. |
| **Inline comments** | **Must fix only** — label + 1–2 sentences + fix |

**Chat summary (default):**

```markdown
**Can we merge?** Yes | No

| | |
|---|---|
| **Must fix** | … or None |
| **Optional** | (1) … or None — not blocking |
| **After merge** | Deploy …; smoke … |
```

- Skip “what looks good” unless the user asks why you approved.
- No history banners (“supersedes review on …”) — one line in **After merge** if re-review matters.

---

## What you need from the user

| Parameter | Required? | Meaning |
|-----------|-----------|---------|
| **`repo`** | Yes (for GitHub) | e.g. `owner/repo` |
| **`pr`** | PR **or** branch | PR number or URL |
| **`branch`** | PR **or** branch | Branch name if no PR yet |
| **`docs`** | Recommended | Paths or links to spec / plan / requirements / ACs |
| **`post`** | No | `true` = post on GitHub · `false` = report only (default) |

If `docs` is missing, discover them from the PR body, linked issues, `docs/`, `README`, tickets, or ask once for the agreement documents.

---

## Docs to read first

Read whatever exists for **this** change. Typical sources (any mix is fine):

| Doc type | Examples | Why |
|----------|----------|-----|
| Spec / contract | OpenAPI, ADR, API contract, design doc | Agreed shapes and behavior |
| Plan | Phase plan, implementation notes, ticket description | What must be built |
| Requirements | PRD, REQUIREMENTS.md, user stories | Product rules |
| Acceptance criteria | Given–When–Then, checklist in ticket | Pass/fail for the workflow |
| Test plan | VALIDATION, CI notes, test list | What should pass |
| Conventions | CONTRIBUTING, style guide, TESTING.md | Team norms |

If a doc is still marked **DRAFT**, warn the requester — do not treat it as final.

More path patterns: see [reference.md](reference.md).

---

## Related user workflows

Check that the **user workflow** still works — not only that APIs or types match.

| Source | What you get |
|--------|--------------|
| Acceptance criteria / user stories | Happy path + edges |
| Requirements / PRD | Actor rules, out-of-scope |
| Plan / ticket | Must-haves vs nice-to-haves |

**Pass B checks:** happy path, edge cases, actor rules, out-of-scope creep. Cite **story / AC / REQ ID** in findings when available.

---

## Steps

### 1. Find the change

```bash
gh pr view <N> --repo <owner/repo> --json title,body,headRefName,baseRefName,headRefOid,files,additions,deletions,state,url
gh pr diff <N> --repo <owner/repo>
```

Branch only:

```bash
git fetch origin
git log origin/<base>..origin/<branch> --oneline
git diff origin/<base>...origin/<branch>
```

### 2. Open the local code

Checkout the PR head or branch in the local clone. Do not stash dirty work without asking.

```bash
git fetch origin <headRefName> && git checkout <headRefName>
```

### 3. Read agreement docs

At least: stated goal + any spec/plan/requirements/ACs linked for this change. Optional: conventions and test docs.

### 4. Review code (four passes)

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
| **After merge** | After merge | Deploy steps, smoke tests, ops notes (not PR blockers) |

### 5b. Merge decision

Set **Can we merge?** from **Must fix** only:

| **Must fix** | **Can we merge?** |
|--------------|-------------------|
| `None` | **Yes** |
| Has items | **No** |

Optional items never change this. Do not use "Yes, after fixes" or "Approved with notes".

### GitHub approve checkbox (required)

When **`post: true`**, the review body **and** the GitHub review event must match:

| **Can we merge?** | GitHub `event` | Reviewers UI |
|-------------------|----------------|--------------|
| **Yes** | `APPROVE` | Green check — counts toward required approvals |
| **No** | `REQUEST_CHANGES` | Changes requested — blocks merge |

**Never** use `COMMENT` when **`post: true`** — that only adds text and does **not** check Approve, even if the body says "Can we merge? Yes".

Use `COMMENT` only when the user explicitly asks for feedback without approving or blocking (e.g. `post: false`, or "comment only, don't approve").

### 6. Share the result

- **`post: false`** → chat **compact summary** (see [Keep it short](#keep-it-short-required))
- **`post: true`** → post short GitHub body with the correct **`event`**; **inline comments only for Must fix**; still give the user the compact chat summary

Pick `event` from the verdict. Full `gh api` examples: [reference.md](reference.md).

Shortcut when **Can we merge?** is **Yes**:

```bash
gh pr review <N> --repo <owner/repo> --approve --body "$(cat <<'EOF'
## PR review — …
**Can we merge?** Yes
…
EOF
)"
```

- **Yes** → always `APPROVE` (or `gh pr review --approve`)
- **No** → always `REQUEST_CHANGES`
- Omit inline comments when **Must fix** is `None`
- One finding per inline comment; Must fix only
- Link spec / plan / AC / REQ when useful — still keep it short

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

- **Must fix = None** ↔ **Can we merge? = Yes** ↔ **`event: APPROVE`**
- **Must fix has items** ↔ **Can we merge? = No** ↔ **`event: REQUEST_CHANGES`**

Hard limits:

- **≤25 lines** for the posted body
- One bullet = one finding
- No inline comments for Optional items

---

## Cross-repo / dependency notes

If the change depends on another PR or migration:

1. Call out the dependency in **After merge** or **Must fix** (Must fix only if this PR is unsafe alone).
2. Prefer a sensible order: schema/migrations → backend → clients (FE/mobile), unless the team’s docs say otherwise.

---

## When **not** to use

| Situation | Do this instead |
|-----------|-----------------|
| No PR/branch and no agreed goal | Ask for scope first |
| Author fixing review threads | Reply/fix workflow for that team |
| Security deep-dive only | Threat-model / security skill if available |
| Local cleanup with no merge decision | Refactor / simplify skill if available |
