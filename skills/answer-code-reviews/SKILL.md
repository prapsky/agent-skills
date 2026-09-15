---
name: answer-code-reviews
description: >-
  Handles GitHub PR review feedback end-to-end: reads comments, fixes code
  against requirements and acceptance criteria, commits and pushes (when asked),
  replies to each finding, and resolves threads.
  Trigger on: answer code reviews, address PR feedback, fix review comments,
  respond to review, resolve review threads, reply to reviewer, fix what the
  reviewer said, address findings, handle PR comments, push fixes and reply,
  reply to each finding, go through review feedback, tackle review comments,
  fix review, answer the reviewer.
  Not for writing a formal merge verdict → use code-review instead.
  Not for local cleanup with no review threads → fix in place and open a PR later.
---

# Answer code reviews

## What this skill does

- Reads feedback on an **existing** GitHub PR.
- Loads **related agreement docs** when the PR links to them (spec, plan,
  requirements, ACs).
- Fixes code and tests where needed.
- Commits and pushes to the PR branch **when asked**.
- Replies to **every** finding.
- Resolves threads **when asked**.

**Analogy:** A review is a punch list after inspection. Walk each item, fix it,
leave a short note, and mark it done.

**Pair skill:** [`code-review`](../code-review/) **writes** the formal review.
This skill **responds** to it (and to any other reviewer feedback).

---

## Keep it short (required)

Summarize the review first. Do not paste or restate the full review body.

| Moment | What to say |
|--------|-------------|
| **Before fixing** | Compact table: Verdict · Must fix · Optional · Do next |
| **After fixing** | Short results table: Finding · Action · Reply link/SHA |
| **GitHub replies** | 1–3 sentences per finding (templates below) |

**Chat summary template (before work):**

```markdown
**Verdict:** Approved | Changes needed | Notes only

| | |
|---|---|
| **Must fix** | … or None |
| **Optional** | (1) … (2) … or None — not blocking |
| **Do next** | Fix / follow-up commit / ask user … |
```

- Expand a finding only when the user asks, or when the fix needs a decision.
- Skip restating "what looks good," deploy essays, and historical notes unless
  they change the action.

Map older labels to current ones before working:

| Incoming label | Treat as |
|----------------|----------|
| Must fix / BLOCKER / CRITICAL / REQUEST_CHANGES item | **Must fix** |
| Should fix / Optional / NIT / Minor | **Optional** (fix if cheap) |
| Note only / OBSERVATION / After merge | Acknowledge; fix only if easy and asked |

---

## Related user workflows

Review comments are about code — the real test is: **does the user workflow
still work?**

| Source | What you get |
|--------|--------------|
| Acceptance criteria | Pass/fail for the flow |
| Requirements / PRD | Product rules |
| Spec / contract | Agreed shapes |
| Plan / ticket | Must-haves |

Discover: PR body → linked issues/docs → `docs/` / `specs/` paths named in the
ticket. Grep REQ / story IDs from the finding. When fixing, map finding → AC →
confirm the workflow still passes. Do not "fix" a test by breaking a business
rule.

---

## What you need

| Parameter | Required? | Meaning |
|-----------|-----------|---------|
| **`pr`** | Yes | PR number, URL, or `owner/repo#N` |
| **`repo`** | No | GitHub slug if not clear from URL |
| **`commit`** | No | `true` when user asks to commit |
| **`push`** | No | `true` when user asks to push |
| **`reply`** | No | `true` by default — post replies |
| **`resolve`** | No | `true` when user asks to resolve threads |

**Examples:**

```text
Use answer-code-reviews on https://github.com/acme/payments/pull/88
```

```text
/answer-code-reviews pr=88 repo=acme/payments commit push reply resolve
```

---

## Steps

### 1. Collect the feedback

```bash
gh pr view <N> --repo <owner/repo> \
  --json title,body,headRefName,baseRefName,state,comments,reviews,headRefOid,files

gh api repos/<owner>/<repo>/pulls/<N>/comments \
  --jq '.[] | {id, path, line, body, user: .user.login, in_reply_to_id}'
```

Thread state (only if inline comments exist):

```bash
gh api graphql -f query='
query {
  repository(owner: "<owner>", name: "<repo>") {
    pullRequest(number: <N>) {
      reviewThreads(first: 100) {
        nodes { id isResolved path line comments(first: 1) { nodes { body } } }
      }
    }
  }
}'
```

| Shape | How you know | How you reply |
|-------|--------------|---------------|
| **Inline threads** | Comments on lines | Reply under each root comment |
| **Body-only** (common from `code-review`) | Numbered items; no inline comments | One PR comment covering `[1]`, `[2]`, … |

Parse `### [1]`, severity labels, and **File:** lines. Each section = one
finding.

Show the user the **compact summary** before starting fixes.

### 2. Open the right local repo

```bash
cd "<local-repo>"
git fetch origin <headRefName>
git checkout <headRefName>
git pull origin <headRefName>
```

- Do not stash dirty work without asking.
- Stage **only** files for this review fix.

### 3. Fix the code

- Match nearby style; keep diffs small.
- When the reviewer listed options A/B/C, pick the safest small change and
  **name it** in the reply.
- Watch mocks/fakes — tests that mock fields the real code never sets are fake
  passes.
- Add or update tests when the review points at a gap.
- Run focused tests for the touched area.

Re-check spec and acceptance criteria when those docs exist.

### 4. Commit and push (when asked)

Only when the user asked. Follow the repo's commit message rules; otherwise one
clear sentence on *why*.

```bash
git add <files>
git commit -m "$(cat <<'EOF'
Short why-focused message about the review fix.
EOF
)"
git push -u origin <headRefName>
```

Save the commit SHA — every reply should mention it.

### 5. Reply on every finding

**Inline:**

```bash
gh api --method POST repos/<owner>/<repo>/pulls/<N>/comments \
  -f body='<reply markdown>' \
  -F in_reply_to=<comment_id>
```

**Body-only — one PR comment:**

```bash
gh pr comment <N> --repo <owner/repo> --body "$(cat <<'EOF'
Thanks — addressed in `<short-sha>`.

### [1] <title>
Fixed. <what changed>. <test note>.

### [2] <title>
Acknowledged — kept as-is.
EOF
)"
```

**Reply templates:**

```markdown
Fixed in `<short-sha>`. <one sentence>. <test note if needed>.
```

```markdown
Acknowledged. <follow-up or plan that covers it>.
```

```markdown
Leaving as-is: <short reason>. Happy to revisit if <condition>.
```

Reply to **every** finding. Be specific but short.

### 6. Resolve threads (when asked)

Only **inline** threads can be resolved. Resolve after push + replies.

```bash
gh api graphql -f query='
mutation {
  resolveReviewThread(input: {threadId: "<PRRT_...>"}) {
    thread { id isResolved }
  }
}'
```

---

## Checklist

```
- [ ] Fetched PR + comments + review bodies
- [ ] Gave user compact summary (not full review dump)
- [ ] Loaded agreement docs if linked
- [ ] Classified findings (Must fix / Optional / Note)
- [ ] Checked out PR branch in the runtime repo
- [ ] Fixed + focused tests pass
- [ ] Committed / pushed if asked
- [ ] Replied on every finding
- [ ] Resolved inline threads if asked
```

---

## What to tell the user (after)

Short table only:

| Finding | Action | Reply |
|---------|--------|-------|
| `[1] <file> — <title>` | Fixed / Acknowledged | link or SHA |

Also: commit SHA, branch, local repo path, PR URL, resolved thread count (or
`N/A — body-only`).

---

## References

Doc discovery tips, focused test commands for Go/JS/Python, and reply hygiene
rules: [references/reference.md](references/reference.md).
