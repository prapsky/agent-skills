# code-review — reference

Companion to [SKILL.md](SKILL.md). Use for posting examples and optional doc discovery.

## Example invocation

```text
Review PR 42 in owner/my-service against docs/specs/api.md and the ticket ACs. post=true
```

```text
Review branch feature/login (base main) in ./my-app. post=false
```

## Doc discovery patterns

When `docs` is not passed, look for:

| Pattern | Typical locations |
|---------|-------------------|
| Spec / contract | `docs/`, `specs/`, `openapi.yaml`, `contracts/` |
| Plan | PR body, linked issue, `PLAN.md`, `.planning/` |
| Requirements | `REQUIREMENTS.md`, `prd.md`, product brief |
| Acceptance criteria | Ticket description, `*acceptance*`, Given–When–Then in docs |
| Tests | `TESTING.md`, CI config, `*VALIDATION*` |
| Conventions | `CONTRIBUTING.md`, `CONVENTIONS.md`, style guides |

Prefer documents **linked from the PR** over random repo docs.

## Pass checklist (detail)

| Pass | Fail → usually |
|------|----------------|
| **A** Agreement | Wrong route/field/status; ignores stated contract |
| **B** Workflow | Missing must-have; breaks actor rules; scope creep without note |
| **C** Quality | Required tests missing/failing; CI red without explanation |
| **D** Risk | Secrets in diff; missing authz; unsafe migration; silent data loss |

## Posting with `gh api`

### Approve (`Can we merge?` Yes)

```bash
gh api --method POST repos/<owner>/<repo>/pulls/<N>/reviews --input - <<'EOF'
{
  "commit_id": "<headRefOid>",
  "event": "APPROVE",
  "body": "## PR review — feature @ `abc1234`\n\n**Can we merge?** Yes\n\n### Must fix\nNone\n\n### Optional\n_Not required before merge._\nNone\n\n### After merge\n- Deploy: service-x\n- Smoke: hit GET /health and one happy-path call"
}
EOF
```

### Request changes (`Can we merge?` No) + Must fix inline comments

```bash
gh api --method POST repos/<owner>/<repo>/pulls/<N>/reviews --input - <<'EOF'
{
  "commit_id": "<headRefOid>",
  "event": "REQUEST_CHANGES",
  "body": "## PR review — feature @ `abc1234`\n\n**Can we merge?** No\n\n### Must fix\n- Auth check missing on DELETE handler\n\n### Optional\n_Not required before merge._\n- Add a unit test name for empty body\n\n### After merge\n- Deploy: after Must fix lands\n- Smoke: DELETE as non-owner should be 403",
  "comments": [
    {
      "path": "internal/api/handler.go",
      "line": 42,
      "side": "RIGHT",
      "body": "**Must fix — missing authz**\n\nDELETE runs without an ownership check. Return 403 when the caller is not the owner."
    }
  ]
}
EOF
```

## Chat vs GitHub

| Channel | Length | Inline comments |
|---------|--------|-----------------|
| Chat | 4-row table only (default) | N/A |
| GitHub body | Same 4 sections, ≤25 lines | Must fix only when posting |

## Mapping from company-specific reviews

If you previously used a team-specific review skill (planning folders, phase numbers, fixed repo maps):

| Old idea | Use here |
|----------|----------|
| Phase contract | Spec / contract path in `docs` |
| PLAN / VALIDATION | Plan + test plan paths |
| Business-process HTML | Acceptance criteria / workflow docs |
| Fixed sibling repo table | Local clone path the user gives |
| Workstream + phase params | Optional `docs` paths + ticket links |

Keep the **same 4-section verdict rules** — that is the reusable core.
