---
title: "Spec-driven development: getting unstuck from vibe coding"
date: 2026-06-12T09:00:00-03:00
draft: false
translationKey: spec-driven-development-sair-do-vibe-coding
category: tecnologia
tags: [spec-driven-development, vibe-coding, ai-tooling, ai-agents, architecture]
author: lucas-israel
description: "Why vibe coding stalls the moment you try to scale — and how spec-driven development delivers an order of magnitude less rework with AI agents."
cover: covers/spec-driven-development-sair-do-vibe-coding.png
cover_alt: "Specification as the central artifact of an AI workflow: the agent reads the versioned spec and generates code from it, instead of guessing from a loose prompt"
cta:
  servico: discovery
  posicao: ambos
  contexto: "Got an idea that's stuck, or a vibe-coded MVP that needs to become a real product? A Discovery turns it into an executable spec — roadmap + prototype in 2-3 weeks."
canonical: ""
---

## TL;DR

You prototyped fast with AI. Now the app won't scale, and every new feature breaks two old ones. That's the exact point where vibe coding stops helping — and where <dfn>spec-driven development (SDD)</dfn> starts paying off. The idea is simple and it flips the order of the game: the **specification becomes the primary artifact**, and the agent implements from it instead of guessing. The trade-off is real — you swap the rush of "it worked first try" for 30 minutes writing a spec before you code. For a throwaway prototype, it's not worth it. For what's going to production and needs to grow, it's what separates delivery from duct tape.

Vibe coding is great for figuring out what to build. It's terrible for sustaining what already exists.

## Vibe coding doesn't fail because it's AI. It fails because it's ambiguous.

In a loose prompt, the model has 30 ways to implement the same feature — and run the same instruction twice, you get something different. That ambiguity is tolerable in a prototype and fatal in maintenance: nobody — not you, not the next dev, not the agent — knows what the rule was. The code is the only source of truth, and it changes with every generation.

If you're a CTO and still on vibe coding, the symptom is familiar: the MVP shipped in a week, the team doubled its speed at first, and now every AI PR needs three rounds of review because the agent "forgot" a decision that was never written down anywhere. I've watched this up close more than once — people think it's a matter of hiring one more senior. It isn't.

**The bottleneck stopped being writing code. It became aligning context.**

## What spec-driven development changes

SDD puts the specification **before** code generation: requirements, business rules, API contracts, and architecture constraints become a document the agent reads and follows. The spec is versioned, reviewed, and reused — the code becomes output, not the source of truth. Less guessing, fewer "that's not what I meant" loops.

In practice the flow is straightforward: you describe the behavior and the constraints → the agent proposes a plan against the spec → you validate the plan (not 400 lines of diff) → the agent implements and tests against the criteria the spec itself defined. Review stops being "is this right?" and becomes "does this match the spec?" — a question you can answer in minutes.

This isn't blog theory. On internal projects with [Spec Kit, GitHub reports roughly an order of magnitude fewer "regenerate from scratch" cycles](https://www.turingpost.com/p/sdd) than ad-hoc prompting. AWS documents, with Kiro, [cases of 40-hour features delivered in under 8 hours of human time](https://towardsdatascience.com/from-vibe-coding-to-spec-driven-development/) when the work started from the spec. And the very person who coined "vibe coding", Andrej Karpathy, has publicly acknowledged the limits of the approach for real software.

### Does SDD work with AI agents like Claude and Copilot?

Yes, and that's exactly what it was built for. Tools like GitHub Spec Kit and AWS Kiro integrate with agents like Claude Code, Copilot, and Gemini CLI. The spec becomes the context the agent follows — the same role a well-written `CLAUDE.md` plays day to day, just promoted to a first-class artifact of the project.

{{< cta servico="discovery" variante="mid" />}}

## Where this breaks

SDD is no silver bullet, and pretending it is would be falling for the same mistake as the vibe coding hype.

> "This is just more ceremony. One more pretty document nobody reads."

It can be. That's the real risk, and I've watched it turn into exactly that. Writing a spec costs head time — for a one-day spike, a hypothesis test, or a throwaway, the overhead doesn't pay off: vibe coding wins. SDD is also only as good as the spec: a vague spec produces vague code, and you've only moved the ambiguity into a prettier document.

The rule I use is simple: **if the code will live longer than a month or pass through someone else's hands, spec it. If it's throwaway, don't.** The decision is by stage, not by dogma.

### Does spec-driven development replace vibe coding?

Not for everything. Vibe coding stays great for prototypes, spikes, and hypothesis validation — where the speed of discovering beats the discipline of sustaining. SDD wins when the code goes to production, needs to scale, or passes through other people's hands. It's not one replacing the other. It's knowing which stage you're in.

## How to get out of vibe coding without stopping the team

You don't have to rewrite everything. The migration is incremental and starts on the next feature, not in a big bang:

- **Write the spec before calling the agent** — even a short one. Expected behavior, rules, constraints. Five lines already change the game.
- **Every business rule lives in the spec** — not in a comment, not in Slack, not in someone's head. If it's not in the spec, it doesn't exist to the agent.
- **Use the spec as the review criterion** — the question stops being "is this good?" and becomes "does this match what we specified?".

Within a few weeks the rework drops, because the context stopped evaporating between one generation and the next.

### Does SDD make development slower?

At the start of each feature, yes — you invest those 30 minutes writing the spec. Overall, it tends to be faster: it's the difference between the order of magnitude fewer regenerate-from-scratch cycles GitHub reports and the 40-hour features delivered in under 8 hours AWS reports. You pay upfront so you don't pay the compound interest of rework later.

## The spec is the context that doesn't evaporate

Vibe coding gives you the first mile for free and charges the rest of the road in rework. SDD does the opposite: it charges upfront and gives back predictability.

The point isn't to abandon AI — it's to stop treating the generated code as the source of truth. The source of truth is the spec. The code is just the output.

If your team is going to burn AI either way, burn it on what's specified.
