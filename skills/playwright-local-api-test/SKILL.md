---
name: playwright-local-api-test
description: >-
  Run local Playwright tests for UI / browser flows only. Do not use this skill
  for HTTP API checks — use python-local-api-test instead. Use when the user
  asks for Playwright UI E2E, screenshots, or browser automation.
---

# Playwright — UI test (not for API)

## Hard rule

**Local HTTP API testing → `python-local-api-test`.**  
This skill is for **UI / browser** automation only.

## When to use / skip

| Use | Skip |
|-----|------|
| UI E2E, page flows, screenshots | Local HTTP API → `python-local-api-test` |
| Browser automation the user explicitly asks for | SQL seed → `local-docker-mysql` |

## If the user asks for local API testing

Redirect immediately to **`python-local-api-test`**. Do not run Playwright for HTTP API probes.

## Notes

Legacy folder name `playwright-local-api-test` remains for compatibility; treat as UI-oriented.

When UI flows **create** phones or string IDs, follow [`test-data-conventions`](../test-data-conventions/SKILL.md) (`+6285YYMMDDxxx`, valid UUIDs).
