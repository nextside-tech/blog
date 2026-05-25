---
title: "ADR (Architecture Decision Record)"
description: "ADR without bureaucracy: log the technical decision in 3 fields on Notion, dated, read by human and AI. No corporate template."
---

An ADR is a technical decision record. Short. Dated. Immutable. You decide something important today, write down why you decided it, and six months from now the answer is right there — no need to dig up the meeting buried in February's calendar.

The point isn't the template. The point is not losing history. Most teams copy Michael Nygard's classic 6-section template, write 3 ADRs in 4 days, and abandon it on the fifth. A small team has no patience for auditor ritual. That's why the Nextside format is leaner: title, status, date, three body paragraphs. Done.

Documenting everything is the same as documenting nothing — it becomes noise. The trigger rule is objective: if the decision costs 1+ day to reverse, write an ADR. If it fits in a 50-line PR, write nothing.

## Why this topic matters

ADRs leveled up when AI entered the flow. Before, the document served the new human joining the project six months later. Now it also serves the Claude Code that opens the repo from scratch every session — and that consults the ADR INDEX before proposing any architectural decision.

As we showed in [ADRs on Notion, without bureaucracy](/en/posts/2026/05/16/adrs-decisao-no-notion-sem-burocracia/), the real value of the ADR isn't in the document. It's in the act of writing — which forces clarity on a decision that was implicit and poorly shaped. The lightweight INDEX, read by the AI, guarantees that clarity reaches the next implementation without anyone needing to remember.

## Tagged posts
