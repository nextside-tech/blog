---
title: "MCP (Model Context Protocol)"
description: "MCP — Model Context Protocol: Anthropic's open protocol that wires LLMs to external tools. USB for AI."
---

MCP is the Model Context Protocol — an open protocol created by Anthropic to wire LLMs to external tools. Think USB for AI: one standard, plug it in, and any compatible client (Claude Code, Claude Desktop, Cursor) talks to any MCP server on the market.

Before MCP, integrating AI with an external tool was artisanal. Each client had its own way to invoke tools. Each tool needed a specific adapter. Chaos. The protocol fixes that with three simple pieces: client (where the AI runs), MCP server (process that exposes tools), and tools (what the server offers — "navigate to URL", "read this file", "run this query").

Today there's an MCP server for pretty much everything: GitHub, Linear, Notion, Postgres, browser via Playwright, filesystem, Slack. You plug in what you need, and the AI operates those tools as an extension of the client itself.

## Why this topic matters

The focus here is on what you can use now, not on the hype. MCP Playwright validates frontend locally before the PR. MCP Notion queries the knowledge base without you opening the browser. MCP Postgres lets the agent run a diagnostic query against the dev database. All standardized, all pluggable.

The Nextside angle: the value of MCP isn't each individual server — it's stopping the reinvention of artisanal integrations. When the protocol is standard, effort goes into solving the real problem, not into making the AI talk to the right tool.

## Tagged posts
