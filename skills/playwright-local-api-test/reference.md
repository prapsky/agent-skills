# Reference — Playwright local API test

## `result.html` template

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Playwright local API result</title>
  <style>
    body { font-family: system-ui, sans-serif; max-width: 800px; margin: 2rem auto; padding: 0 1rem; }
    .ok { color: #0f7b3a; font-weight: 700; }
    .fail { color: #b00020; font-weight: 700; }
    table { width: 100%; border-collapse: collapse; }
    th, td { text-align: left; padding: .6rem .4rem; border-bottom: 1px solid #eee; vertical-align: top; }
    th { width: 30%; color: #555; }
    code { background: #f2f2f2; padding: .1rem .35rem; border-radius: 4px; }
    pre { background: #f7f7f7; padding: 1rem; overflow: auto; border-radius: 8px; }
  </style>
</head>
<body>
  <h1>Playwright local API result</h1>
  <p class="ok">PASSED</p><!-- or class="fail">FAILED -->
  <table>
    <tr><th>Endpoint</th><td><code>POST http://127.0.0.1:5007/v1/example</code></td></tr>
    <tr><th>Seed</th><td><pre>{ "id": "...", "phone": "..." }</pre></td></tr>
    <tr><th>Request</th><td><pre>{ }</pre></td></tr>
    <tr><th>Status</th><td><code>201</code></td></tr>
    <tr><th>Response</th><td><pre>{ }</pre></td></tr>
    <tr><th>Playwright report</th><td><a href="./index.html">index.html</a></td></tr>
    <tr><th>When</th><td>ISO timestamp</td></tr>
  </table>
</body>
</html>
```

## Mapping seed → request

Examples (adapt per endpoint):

| Seed field | Typical use |
|------------|-------------|
| `id` | Path param or body id |
| `phone` | Body `phone` / lookup key |
| `club` / `full_name` | Body `homeclub` / `locationUser` |
| `email` | Body `email` |

Ask the user for the mapping when unclear.

## Pairing with `mysql-insert`

1. `mysql-insert` → user runs / agent runs INSERT + verify SELECT  
2. Capture verify row as seed JSON  
3. This skill → Playwright call + HTML report  

## Optional cleanup

Only if the user asks: run the cleanup `DELETE` from `mysql-insert`, or reset related app state (e.g. clear a flag that blocks re-claims).
