# agent-skills

Production-grade **skills** for AI coding agents (Cursor, Claude Code, Codex, and similar tools).

Think of each skill as a short playbook: when a task matches the skill’s description, the agent reads `SKILL.md` and follows that process instead of improvising.

## What’s in this repo

```text
agent-skills/
├── LICENSE
├── README.md
└── skills/
    ├── answer-code-reviews/
    ├── business-process/
    ├── code-review/
    ├── create-pull-request/
    ├── local-docker-dynamodb/
    ├── local-docker-firestore/
    ├── local-docker-mysql/
    ├── local-docker-redis/
    ├── playwright-local-api-test/
    ├── python-local-api-test/
    ├── test-cases/
    └── unit-test/
```

| Skill | What it helps the agent do |
| --- | --- |
| [`business-process`](skills/business-process/) | Author a standalone non-technical business-process overview HTML (FE↔BE, systems, infra inventory). |
| [`test-cases`](skills/test-cases/) | Author positive/negative QA test cases (HTML or Markdown) from use cases and acceptance criteria. |
| [`unit-test`](skills/unit-test/) | Write or rewrite automated unit tests as table-driven cases (match package style, run focused tests). |
| [`code-review`](skills/code-review/) | Formal PR/branch review against agreed docs, with a clear Yes/No merge verdict (optional GitHub post). |
| [`answer-code-reviews`](skills/answer-code-reviews/) | Address PR review feedback: fix, commit/push, reply on each finding, resolve threads. |
| [`create-pull-request`](skills/create-pull-request/) | Open a GitHub PR with a clear title, Summary, and Test plan (apply relevant skills to the diff first). |
| [`local-docker-mysql`](skills/local-docker-mysql/) | Start / seed (INSERT…SELECT) / verify / inspect local Docker MySQL (`app-mysql-local`). Replaces former `mysql-insert`. |
| [`local-docker-redis`](skills/local-docker-redis/) | Start / health-check / seed keys / scan for local Docker Redis (`app-redis-local`). |
| [`local-docker-dynamodb`](skills/local-docker-dynamodb/) | Start / tables / PutItem / scan for local DynamoDB (`app-dynamodb-local` → `:8000`). |
| [`local-docker-firestore`](skills/local-docker-firestore/) | Start / seed / read for Firestore emulator (`app-firestore-local` → `:8080`). |
| [`python-local-api-test`](skills/python-local-api-test/) | Run local **HTTP API** tests with **Python** against Docker stores; always deliver the standard result report. Prefer this over Playwright for APIs. |
| [`playwright-local-api-test`](skills/playwright-local-api-test/) | **UI / browser** Playwright only — redirect HTTP API work to `python-local-api-test`. |

`local-docker-*` skills are company-agnostic defaults (`app-*-local`). Rename containers/creds per project. `local-docker-mysql` and `python-local-api-test` work together for relational API tests; add Redis/Dynamo/Firestore skills when those stores are in the path. Playwright is for UI only. `code-review` and `answer-code-reviews` are a pair. `business-process` and `test-cases` are a pair. `test-cases` (human checklist) and `unit-test` (automated code tests) are different layers — do not swap them.

## How a skill is structured

Every skill folder follows the same shape:

| File | Role |
| --- | --- |
| `SKILL.md` | Main instructions (YAML frontmatter + workflow). Agents load this first. |
| `reference.md` | Deeper examples and notes. Open when `SKILL.md` points to it. |

Frontmatter at the top of `SKILL.md` tells the agent **when** to use the skill (`name` + `description`).

## How to use these skills

### Option A — Copy into a project

Copy a skill folder into your agent’s skills directory, for example:

```bash
cp -R skills/local-docker-mysql /path/to/your-project/.cursor/skills/
cp -R skills/local-docker-redis /path/to/your-project/.cursor/skills/
cp -R skills/local-docker-dynamodb /path/to/your-project/.cursor/skills/
cp -R skills/local-docker-firestore /path/to/your-project/.cursor/skills/
cp -R skills/python-local-api-test /path/to/your-project/.cursor/skills/
```

Exact install paths depend on the tool (Cursor `.cursor/skills/`, personal `~/.cursor/skills/`, etc.).

### Option B — Point your agent at this repo

Clone or symlink this repository and configure your agent to load skills from `skills/`.

```bash
git clone git@github.com-personal:prapsky/agent-skills.git
# or: git clone https://github.com/prapsky/agent-skills.git
```

## Typical flows (high level)

### Seed data + local API test

```mermaid
sequenceDiagram
  participant User
  participant Agent
  participant MySQL as local-docker-mysql
  participant Redis as local-docker-redis
  participant PW as python-local-api-test skill

  User->>Agent: Need seed data + test this API
  Agent->>MySQL: Start Docker MySQL, then seed
  Agent->>Redis: Start Docker Redis (if cache used)
  MySQL-->>Agent: Seed rows (ids, fields)
  Agent->>PW: Python HTTP probe (Docker MySQL + Redis)
  PW-->>Agent: HTML report + standard result report
  Agent-->>User: Results + report path (+ PR comment if PR in context)
```

### Multi-store local preflight

```mermaid
flowchart LR
  U[User: test locally] --> M[local-docker-mysql]
  U --> R[local-docker-redis]
  U --> D[local-docker-dynamodb]
  U --> F[local-docker-firestore]
  M --> API[Local API / Playwright]
  R --> API
  D --> API
  F --> API
```

### Code review + answer feedback

```mermaid
sequenceDiagram
  participant User
  participant Agent
  participant Review as code-review skill
  participant Answer as answer-code-reviews skill
  participant GH as GitHub

  User->>Agent: Review this PR against the plan
  Agent->>Review: Read docs + diff (four passes)
  Review-->>Agent: Yes/No + Must fix / Optional / After merge
  opt post=true
    Agent->>GH: APPROVE or REQUEST_CHANGES
  end
  Agent-->>User: Compact 4-section verdict
  User->>Agent: Address the review comments
  Agent->>Answer: Fix, push, reply, resolve
  Answer->>GH: Replies (+ optional resolve)
  Agent-->>User: Finding / Action / Reply table
```

1. **You** ask for seed SQL, a local API check, a business-process HTML overview, QA test cases, unit tests, a PR review, or help answering review comments.
2. **Agent** matches the request to a skill and reads `SKILL.md`.
3. **Behind the scenes:** discover data / call APIs / compare to agreement docs / research a journey / draft cases / write table-driven unit tests / fix and reply on threads.
4. **You** get SQL, an HTML report plus the standard local-test result report, a process overview page, a test-case table, passing unit tests, a merge verdict, or a short “what we fixed” table—not a vague essay.

## Adding a new skill

1. Create `skills/<skill-name>/`.
2. Add `SKILL.md` with `name` and `description` frontmatter.
3. Add `reference.md` when examples would clutter the main skill.
4. Update this README’s skill table.

Keep instructions short, safe by default (no production writes or secrets unless the user clearly asks), and easy for a non-expert to follow.

## License

MIT — see [LICENSE](LICENSE).
