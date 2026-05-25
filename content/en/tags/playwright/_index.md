---
title: "Playwright"
description: "Playwright as a local validation tool: Claude drives the browser, takes screenshots, catches regressions before the PR."
---

Playwright is Microsoft's browser automation stack. Headless or not. Cross-browser (Chromium, Firefox, WebKit). Consistent, performant API, with excellent DX. What Selenium wanted to be and never managed.

There are two ways to use it today, and they don't compete. Traditional Playwright is spec in code, versioned in the repo, running in CI on every PR — that's the E2E suite that blocks merge when a real regression shows up. Playwright via MCP is different: Claude gets eyes on the browser, navigates your local app, takes screenshots, reads the DOM, checks the console. Without you writing test specs.

As we explored in [MCP Playwright: local validation with real quality](/en/posts/2026/05/16/mcp-playwright-validacao-local-com-qualidade/), the AI tires less than you do. It doesn't forget to test dark mode. It doesn't skip mobile in a rush. It doesn't say "I'll look at the console later." Every time it runs, it runs everything.

## Why this topic matters

The Nextside angle: Playwright via MCP doesn't replace CI. ABSOLUTELY NOT. It complements. It's step 3 of the dev flow — that manual pre-PR click everyone skips — automated by the agent. The E2E suite in CI is still the insurance that blocks critical regressions. MCP Playwright is the preflight that catches visual bugs before the PR ships.

Whoever swaps the E2E suite for MCP Playwright will miss it on the first big refactor. Whoever uses both gains consistency that a CI-only team never reaches with human willpower.

## Tagged posts
