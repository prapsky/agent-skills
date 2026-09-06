# Business process HTML — reference

Open this when building the **Infrastructure Inventory**, copying the **CSS design system**, or starting from the **HTML skeleton**.

---

## Infrastructure inventory

Exact names from code / IaC / env — not only “a database” or “a message.”

**Analogy:** Address book — mailbox names (messaging), locker numbers (cache), filing cabinets (tables), light switches (flags), street addresses (URLs).

### Collect checklist

| Category | What to write | Where to find it |
|----------|---------------|------------------|
| **Cache (e.g. Redis)** | Key pattern, TTL, service R/W, purpose | Cache helpers, env key prefixes |
| **Messaging (e.g. Pub/Sub)** | Topic, subscription, DLQ, push endpoint, attributes | Publishers, IaC, consumers |
| **Feature flags / remote config** | Exact key, provider, off/on meaning, who checks | App config, BE flags, config docs |
| **SQL** | DB (if relevant), table, columns touched, R/W | Models, migrations, SQL |
| **Document store** | Collection/doc path, ID pattern, fields | Repos, clients |
| **KV / NoSQL tables** | Table, keys/indexes if useful, attributes | Repos, table defs |
| **Object storage** | Bucket/path, object key pattern | Upload code, IaC |
| **Serverless / workers** | Function name, trigger, purpose | Deploy configs, console |
| **Task queues / schedulers** | Queue/job name, delay/schedule, target URL | Task creators |
| **HTTP / URLs** | Full URL or `BASE + path`, method, caller → callee | Gateway, FE env, Postman |

### Suggested table columns

**Cache**

| Key / pattern | Example resolved key | TTL | Operation | Service | Purpose |

**Messaging**

| Topic | Subscription | Push endpoint | DLQ | Channel / when | Attributes |

Also note publisher service and a plain-language payload summary (ids, phone, etc.) — not a full schema dump.

**Feature flags**

| Flag / config key | Provider | Default / off | On behavior | Checked by | Notes |

**SQL**

| Database (optional) | Table | Columns / fields | Read or Write | When |

**Document store**

| Collection / doc path | Document ID pattern | Fields | Read or Write | When |

**KV / NoSQL**

| Table | PK / SK (if helpful) | Attributes | Read or Write | When |

**Object storage**

| Bucket (or logical path) | Object key pattern | Content | Channel |

**Workers**

| Function / job name | Trigger or schedule | Invokes / target | Purpose | Process step |

**HTTP**

| Caller | Environment | Method | Full URL or `BASE + path` | Auth | Purpose |

### URL rules

1. Method badge everywhere (`GET` / `POST` / …).
2. Path in Endpoints section; full addresses in Infrastructure.
3. Use concrete hosts **or** env formulas (`{API_GATEWAY_BASE_URL}/v1/…`) and name the env var.
4. List FE page URLs users open and push-consumer URLs for background jobs.
5. Prefer public/gateway paths in the endpoints section; put raw service hosts in inventory if engineers need them.
6. Never paste secrets.

```html
<p class="note">
  Hosts differ by environment. Replace
  <code>{API_GATEWAY_BASE_URL}</code>,
  <code>{WEB_BASE_URL}</code>,
  <code>{ADMIN_BASE_URL}</code>
  with staging or production values from each app’s <code>.env</code>.
</p>
```

### Plain-language intros (before each table)

- **Cache:** “Fast temporary memory — these are the exact locker labels.”
- **Messaging:** “Internal mail — topics are mailboxes; subscriptions pick up the mail.”
- **Flags:** “Remote switches that turn parts of the process on or off.”
- **Tables:** “Filing cabinets for permanent records.”
- **Workers / tasks:** “Background workers that run after the user has moved on.”
- **URLs:** “Full addresses browsers and services call (hosts change by environment).”

### Where to place tables

| Content | Place |
|---------|-------|
| Method + path + plain meaning | Endpoints section |
| Topic → subscription → push → DLQ | Infrastructure (or inside data section) |
| Storage names + channel matrix | Data-change section |
| Cache, flags, workers, full URLs | Infrastructure Inventory |
| Mentions in FE↔BE | Inline in backend cells |

---

## CSS design system

Reuse one shared palette across process pages. Do not invent a new look per page.

```css
--bg: #eef1f6;
--card: #fff;
--text: #12151c;
--muted: #5c6370;
--border: #d8dde6;
--accent: #0d47a1;
--accent-2: #7c3aed;
--accent-soft: #e3ecfc;
--warn: #b45309;
--ok: #166534;
--radius: 14px;
```

| Class | Use |
|-------|-----|
| `header` + `eyebrow` + `tags` | Hero band |
| `.wrap` | Max width ~1040px |
| `.card` | Section container |
| `h2 .num` | Numbered section badge |
| `.lead` | Section intro |
| `.analogy` | Analogy callout |
| `.steps` / `.step` | High-level stages |
| `.pill` (+ channel modifiers) | Channel / system badges |
| `.method.get` / `.post` | HTTP method badges |
| `.flow-table` | FE↔BE paired steps |
| `.diagram-card` + `pre.mermaid` | Diagrams |
| `.surface` + `.grid-2` | Impact cards |
| `.takeaways` | Final green summary |
| `.store-pill` / `.data-matrix` / `.store-card` | Data sections |

**Channel pills:** define modifiers that match the project’s channels (e.g. web / admin / app / backend). Colors should stay consistent across pages.

**Storage pills (optional):** distinct classes per store type (SQL, document DB, KV, object storage, cache).

Responsive: stack grids under ~760px; horizontal scroll on wide tables.

If the project already has a canonical process HTML, **copy its `<style>` block** instead of reinventing CSS.

---

## Mermaid loader

```html
<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>
  mermaid.initialize({ startOnLoad: true, theme: "default", flowchart: { curve: "basis" } });
</script>
```

Flowchart: subgraphs for Channels · Core systems · Business outcome (e.g. channels blue, core purple, outcome green).

---

## Minimal HTML skeleton

```html
<!DOCTYPE html>
<!-- {Process} — Business Process Overview (non-technical) -->
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>{Process} — Business Process Overview</title>
  <style>/* shared CSS tokens + layout from an existing process page or this reference */</style>
</head>
<body>
  <header><!-- eyebrow, h1, pitch, tags --></header>
  <div class="wrap">
    <!-- 1 What is it -->
    <!-- 2 High-level stages -->
    <!-- 3 Mermaid flowchart -->
    <!-- 4 Systems FE + BE -->
    <!-- 5 Endpoints by channel -->
    <!-- 6 FE ↔ BE flow tables -->
    <!-- 7 Impact cards -->
    <!-- 8+ Data changes -->
    <!-- Infrastructure inventory -->
    <!-- Last: Key takeaways -->
    <footer>…</footer>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <script>
    mermaid.initialize({ startOnLoad: true, theme: "default", flowchart: { curve: "basis" } });
  </script>
</body>
</html>
```

**Do not** add a Test Cases block — that belongs in a separate skill/deliverable.

---

## Adaptation map (any process)

| Concept | Generalize to |
|---------|---------------|
| Funnel stages | Your process stages |
| Create vs claim (or similar) | Core record write vs full journey |
| Channels | Only surfaces that participate |
| Source-of-truth table | Your primary store |
| Event + subscriptions | Your messaging graph |
| Remote config / flags | Your exact keys |
| Delayed jobs | Your workers / task queues |
| `{API_BASE}/…` | Your full HTTP addresses per env |
| Success metric | Conversion / completion the business cares about |
| Gate rules | Who may proceed (eligibility, auth, quotas) |
)
