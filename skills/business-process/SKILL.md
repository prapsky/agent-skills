---
name: business-process
description: >-
  Author a standalone, non-technical business-process overview as a single
  self-contained HTML file (CSS + Mermaid). Use when the user asks for a
  business process page, process overview HTML, FE↔BE journey doc, use-case
  flow, or infrastructure inventory for how a product journey works.
---

# Business process HTML

## When to use

- User wants a **business process overview** as a standalone `.html` file
- Explaining an end-to-end journey across channels (web, admin, mobile, partner, etc.)
- Documenting use cases, FE↔BE steps, systems, endpoints, and infrastructure names in plain language

## When not to use

- Writing **test cases / QA scripts** for the process → use a separate test skill when available
- OpenAPI dumps, design docs, or code walkthroughs as the main deliverable
- Editing wiki/markdown notes only (unless the user also wants the HTML overview)

## Goals

1. One **self-contained** HTML file (inline CSS + Mermaid) that opens in a browser with no build step.
2. Audience: **non-technical** stakeholders (business, ops, QA, new engineers).
3. Reader understands the funnel, systems, APIs, FE↔BE steps, data writes, and real infrastructure names — without jargon-first prose.

**Do not** turn this into a technical design doc. Technical names (APIs, tables, services) are allowed only after a plain-language explanation.

**Analogy:** The page is a museum tour of the journey — clear signs and maps, not the machine shop blueprints.

---

## Inputs

| Input | Required? | Notes |
|-------|-----------|-------|
| Process name / outcome | Yes | One end-to-end business job |
| Channels / surfaces | Yes | Only those that participate (e.g. Website, Admin, App) |
| Source of truth | Recommended | Wiki, PRD, planning docs, tickets |
| Code / config roots | Recommended | Services, gateway, env, IaC for real names |
| Output path | Yes | Ask if missing; default `{process-kebab}-business-process.html` in the team’s docs folder |

---

## Core concepts

```text
Business process
  └── Use case 1
        └── Acceptance criteria (Given–When–Then…)
  └── Use case 2
        └── …
```

| Layer | Answers | Example |
|-------|---------|---------|
| **Business process** | End-to-end business job | Process a payment (check eligibility → save draft → pay) |
| **Use case** | One actor + one goal | Check eligibility by phone |
| **Acceptance criteria** | Pass/fail rules (prefer Given–When–Then) | Draft becomes Paid and member gets confirmation |

**Restaurant analogy:** process = serving a meal; use case = take the order; acceptance criteria = order includes table number.

### Given–When–Then (for acceptance criteria)

```text
Given <starting situation>
When <actor does something>
Then <expected observable result>
And <optional extra result>
```

Scope checklist before writing HTML:

- [ ] Named the **business process** (one outcome)
- [ ] Listed **use cases** (actor + goal)
- [ ] Drafted **acceptance criteria** per use case (GWT preferred)
- [ ] **Do not** add a Test Cases section to this HTML (out of scope for this skill)

---

## Workflow

```
- [ ] 1. Clarify process, channels, output path
- [ ] 2. Research (docs first → code second)
- [ ] 3. Inventory infrastructure names (§ reference)
- [ ] 4. Draft use cases + acceptance criteria
- [ ] 5. Write HTML from required structure
- [ ] 6. Pre-publish checklist
- [ ] 7. Return file path + short summary
```

### 1. Clarify

Ask only if missing: process name, participating channels, output folder, domain/eyebrow label.

### 2. Research

1. Read product/business docs the user points to (wiki, PRD, planning).
2. Confirm against latest code and config in the relevant repos.
3. If docs and code disagree: **trust code for implementation**; note the gap briefly on the page.
4. Ground claims in real routes, APIs, messaging topics, and storage names as of the document date.

### 3. Infrastructure inventory

Collect exact names before writing HTML. Details and table columns: [reference.md](reference.md).

Categories: cache keys, message topics/subscriptions, feature flags, databases/collections, background jobs, full HTTP URLs. If a category does not apply, say so explicitly (e.g. “This process does not use Redis.”).

### 4–5. Write the HTML

Use the **required document structure** below. Prefer copying CSS/markup from an existing good process page in the project if one exists; otherwise use the tokens and skeleton in [reference.md](reference.md).

### 6–7. Finish

Run the pre-publish checklist. Tell the user the path and what channels/use cases the page covers.

---

## File naming

| Item | Convention |
|------|------------|
| File name | `{process-kebab-case}-business-process.html` |
| Location | Team docs folder (e.g. `.planning/docs/`, `docs/processes/`) — ask if unclear |
| Top HTML comment | Process name, “non-technical”, source of truth, month/year |

```html
<!DOCTYPE html>
<!--
  {Process Name} — Business Process Overview (non-technical)
  Standalone explainer. Grounded in {docs / gateway / code} as of {Mon YYYY}.
-->
<html lang="en">
```

---

## Required document structure

Number sections **sequentially** (1, 2, 3…). Skip only when truly N/A; no numbering gaps.

| # | Section | Include |
|---|---------|---------|
| 1 | **What is {Process}?** | Business definition, who it serves, main goal. One **analogy** box. |
| 2 | **The High-Level Business Process** | 4–8 numbered stages. |
| 3 | **High-Level Flow Diagram** | One Mermaid **flowchart** (channels → core systems → outcome) + short caption. |
| 4 | **Systems Involved (Frontend & Backend)** | Two tables: Frontend (who / role) and Backend (responsibility). Channel pills. |
| 5 | **The Endpoints Involved** | Group by channel. Columns: Action · Endpoint (method badge) · plain-language “what it does”. Paths here; **full host URLs** in Infrastructure Inventory. |
| 6 | **Step-by-Step: FE ↔ BE** | Per-channel flow tables (see below). Subflows when a channel has multiple journeys. |
| 7 | **Impact on each channel** | Grid of surface cards: role of each channel + shared outcome. |
| 8+ | **Where Data Changes** (recommended) | Core write vs full journey side effects; storage reference; channel matrix; sequence diagrams. |
| Next | **Infrastructure Inventory** | Cache, messaging, flags, DBs, jobs, full URLs — see [reference.md](reference.md). |
| Last | **Key Takeaways** | 5–8 bullets. Include critical topic / table / flag names. |

**Do not include** a Test Cases section in this deliverable.

### Header (always)

- Eyebrow: `{Product or org} · {Domain}`
- H1: `{Process} — Business Process Overview`
- One-sentence plain-language pitch
- Tags (pills): e.g. `Non-technical friendly`, main funnel idea, key concept

### Footer (always)

```text
{Product or org} · {Domain} — {Process} business process overview · Generated for non-technical stakeholders
```

---

## Writing rules

1. **Correct, simple English.** Prefer “the system checks…” over jargon-first wording.
2. **Define terms once**, then reuse.
3. **Analogy box** after the definition (and again for create vs claim / similar pairs):

   ```html
   <div class="analogy">
     <strong>Simple analogy:</strong> …
   </div>
   ```

4. **Lead paragraph** under every major section: one muted sentence stating what the section answers.
5. **Always pair Frontend and Backend**:
   - Frontend = what the person sees or clicks
   - Backend = validate, authenticate, business rules, read/write data, call services, publish messages, return response
6. Name real systems when helpful, but keep the sentence readable without them first.

---

## FE ↔ BE flow tables (Section 6)

Most important teaching section.

### Legend (once)

- `→` Direct API call
- `⚡` Background message (user does not wait)
- `—` No API (UI-only / client-side)

### Columns

| # | Frontend — what the user/staff does | (link) | Backend — what happens behind the scenes |

### Cell content

**Frontend:** label `Frontend` · bold action · optional detail · optional `Repo · route · file` in mono.

**Backend:** label `Backend` · method badge + endpoint (or topic name) · owning service + what it writes/checks.

### Row classes

| Class | When |
|-------|------|
| (default) | Sync API (`→`) |
| `async` | Background jobs / messaging (`⚡`) |
| `no-api` | UI-only (`—`) |

### Per channel

1. Heading with channel pill + route/module
2. One-line “Who / Goal”
3. Optional subflows (A / B / C)
4. One `flow-table` per subflow

---

## Data-change sections (recommended)

When the process writes durable data:

1. Plain definition of the verb (create / claim / purchase / cancel…)
2. Analogy separating “core write” vs “full journey”
3. Per-channel capability table (Can it? Who? How? Which API?)
4. Storage reference: store type · table/collection · what it holds
5. Master matrix: storage × channel (`✓` write / Read / `—`)
6. Per channel: Mermaid `sequenceDiagram` + step table
7. Infrastructure tables from [reference.md](reference.md)

---

## Mermaid

Load once at the bottom of the page (see skeleton in [reference.md](reference.md)).

| Where | Type |
|-------|------|
| Section 3 | `flowchart TD` — Channels · Core systems · Outcome |
| Data sections | `sequenceDiagram` — UI → service → stores → optional consumers |

Keep diagrams high-level. Put real topic/table names on arrows when helpful. Avoid private function names unless they help engineers find the path.

---

## Pre-publish checklist

- [ ] Page opens standalone in a browser (no build step)
- [ ] Non-technical reader can explain the process in under five minutes
- [ ] Sections numbered without gaps; **no Test Cases section**
- [ ] Every major frontend action has a matching backend explanation
- [ ] Mermaid diagrams render
- [ ] Endpoints use correct method badges and real paths
- [ ] Background jobs marked with `⚡`
- [ ] Infrastructure categories listed with exact names, or explicit “none”
- [ ] Full URLs use method + `BASE + path` (or concrete host); **no secrets**
- [ ] Data matrices match what code actually reads/writes
- [ ] Key takeaways state the business success metric
- [ ] Date / source note is accurate

---

## Security

- Never paste secrets, tokens, passwords, or signed query strings into the HTML
- Prefer env formulas (`{API_BASE_URL}/…`) over real credentials
- Staging/local examples by default; label production hosts clearly if included

---

## Additional resources

- Infrastructure inventory tables, CSS tokens, and HTML skeleton: [reference.md](reference.md)
)
