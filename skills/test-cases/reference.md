# Test cases — reference

HTML snippets and examples for the `test-cases` skill. Keep product/channel names generic unless the project already defines codes.

---

## Summary strip (HTML)

```html
<div class="tc-summary">
  <span><strong>Total</strong> 12</span>
  <span><strong>Positive</strong> 7</span>
  <span><strong>Negative</strong> 5</span>
  <span><strong>WEB</strong> 4</span>
  <span><strong>ADMIN</strong> 4</span>
  <span><strong>APP</strong> 4</span>
</div>
```

Update counts after the table is final so the strip always matches.

---

## Case table (HTML)

```html
<table class="tc-table">
  <thead>
    <tr>
      <th>ID</th>
      <th>Type</th>
      <th>Scenario</th>
      <th>Steps (what to do)</th>
      <th>Expected result</th>
      <th>API / system</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>WEB-01</td>
      <td><span class="pill pos">Positive</span></td>
      <td>Eligible user completes the happy path on the website</td>
      <td>
        <ol>
          <li>Open the feature page while logged out or as a fresh user</li>
          <li>Fill required fields with valid staging data</li>
          <li>Submit</li>
        </ol>
      </td>
      <td>Success confirmation; core record created; optional background job fired</td>
      <td><code>POST /v1/example</code></td>
    </tr>
    <tr>
      <td>WEB-02</td>
      <td><span class="pill neg">Negative</span></td>
      <td>Submit with a required field missing</td>
      <td>
        <ol>
          <li>Open the feature page</li>
          <li>Leave a required field empty</li>
          <li>Submit</li>
        </ol>
      </td>
      <td>Validation error; no durable write</td>
      <td><code>POST /v1/example</code> or UI-only validation</td>
    </tr>
  </tbody>
</table>
```

---

## Type pills (CSS hints)

If embedding beside a business-process page, reuse its palette. Minimal hints:

```css
.pill.pos { background: #dcfce7; color: #166534; }
.pill.neg { background: #ffedd5; color: #b45309; }
.tc-summary {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem 1.25rem;
  margin-bottom: 1rem;
  font-size: 0.95rem;
}
.tc-table { width: 100%; border-collapse: collapse; }
.tc-table th, .tc-table td {
  border: 1px solid #d8dde6;
  padding: 0.5rem 0.65rem;
  vertical-align: top;
  text-align: left;
}
```

---

## Tester tip (copy pattern)

```html
<p class="note">
  <strong>Tester tip:</strong> Use a clean staging account per run.
  Prefer dedicated test phones/emails. Reset or isolate data before negative
  duplicate cases. Confirm feature flags match the scenario (on vs off).
</p>
```

---

## Coverage brainstorm (optional)

When ACs are thin, walk each use case and ask:

| Prompt | Typical case type |
|--------|-------------------|
| Happy path end-to-end? | Positive |
| Missing / invalid input? | Negative |
| Not authenticated / wrong role? | Negative |
| Duplicate submit? | Negative |
| Business gate (ineligible, expired, quota)? | Negative |
| Flag off vs on? | Both |
| Background message / job after success? | Positive (separate ID) |

---

## Standalone page shell

```html
<!DOCTYPE html>
<!-- {Process} — Test cases -->
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>{Process} — Test cases</title>
  <style>/* optional: copy tokens from business-process reference */</style>
</head>
<body>
  <header>
    <p class="eyebrow">{Product} · {Domain}</p>
    <h1>{Process} — Test cases</h1>
    <p>Positive and negative checks for QA. Not a full system design.</p>
  </header>
  <main>
    <!-- tc-summary -->
    <!-- tc-table -->
    <!-- tester tip -->
  </main>
</body>
</html>
```
