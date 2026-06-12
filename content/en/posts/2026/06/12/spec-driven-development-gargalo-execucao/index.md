---
title: "The spec was the easy part. SDD's bottleneck is execution"
date: 2026-06-12T09:00:00-03:00
draft: false
translationKey: spec-driven-development-gargalo-execucao
category: tecnologia
tags: [spec-driven-development, dynamic-workflows, claude-code, context-rot, orquestracao, ia-development]
author: pablo-winter
description: "Spec-Driven Development externalizes intent into markdown, but pushes the pain into execution. Orchestrate with state outside the model's window."
cover: covers/spec-driven-development-gargalo-execucao.png
cover_alt: "Dark editorial Nextside cover: the execution bottleneck of Spec-Driven Development and a spec being run by AI agents"
cta:
  servico: discovery
  posicao: ambos
  contexto: "Going to put agents to work running specs in your team? We validate where AI actually accelerates in a Discovery, before you burn budget."
canonical: ""
---

## TL;DR

Spec-Driven Development solved a real problem: you externalize intent into versioned markdown (PRD, tech spec, task list) and the spec becomes the source of truth. Except nobody tells you the bill for execution. One spec generates dozens of tasks, and running all of them in a single conversation is where the context degrades and you turn into a window manager. The way out the whole industry converged on, from expensive framework to a bash `while` loop, is the same: take state out of the model's window and put it in a file or in code, with a reviewer who is never the one who wrote it. Anthropic's <dfn>Dynamic Workflows</dfn> — where Claude itself writes the script that orchestrates the agents — are one form of this. There are several.

This post is about why execution was the bottleneck all along, and why everyone is arriving at the same two rules.

## Nobody tells you the bill for execution

Spec-Driven Development is simple to describe: you write the intent before the code. PRD becomes tech spec, tech spec becomes a list of atomic tasks, and only then does the agent generate code. GitHub Spec Kit, Amazon Kiro, Tessl, each with its own flavor. The spec is the source of truth, the code is a consequence.

Writing the spec is the easy part.

My last spec generated thirty-something tasks. Hell didn't start there. It started when I had to execute those thirty-something in a single conversation. You run task after task, the window fills up, and somewhere around the twentieth the agent has already forgotten the decision it made itself on the fourth.

This has a name and it's been measured. <dfn>Context rot</dfn> — the drop in model quality as the context grows — was tested by Chroma across 18 models. All 18 degraded, and the degradation starts well before the window is full. The "Lost in the Middle" paper had already shown the same curve: the model loses information buried in the middle of a long context.

The patch the community adopted is to open a clean window per task: fresh context, paste the spec back in, point at the task, execute, repeat. It works against the rot. And it turns you into a copy-paste intern, thirty-something times over.

**The spec was the easy part.**

## The three phases of whoever carries the context

The bottleneck was always the same: someone has to hold the state and consolidate the results while the tasks run. What changed is who carries that weight.

**Phase 1, by hand.** You are the context window. You run task by task, hit `/clear`, reread the spec, hold the state in your head and in the conversation. Goes fine for five tasks. By the thirtieth, you're the bottleneck.

**Phase 2, delegating.** You throw execution at the subagents. It helps. Except the output of all of them comes back into the same window, the one of the main agent you're steering, and that window is the one that becomes the consolidator and rots. Agent Teams got better with a shared task list, but the lead still steers step by step. The bottleneck moved, it didn't disappear.

**Phase 3, workflow.** Here the physics changes. The plan leaves your context and becomes code. A script holds the loop and the intermediate results, and the model's context only sees the final answer. Each task runs in an isolated window. This is where I finally stopped being the bottleneck. It's what Claude Code's Dynamic Workflows do: Claude itself writes a JavaScript orchestration script, and a runtime executes it in the background, with up to 16 simultaneous agents and a ceiling of a thousand per run.

Jarred Sumner, creator of Bun, took this to the extreme. He ported Bun from Zig to Rust on exactly this setup: tasks in parallel, two reviewers contesting each file. Seven hundred and fifty thousand lines of Rust, 99.8% of the test suite passing, eleven days from first commit to merge. It hasn't gone to production yet, it's a capability demo. But that's the number.

### Why can't the reviewer be the author?

Because the model has self-preference bias. Self-preferential bias is the model's tendency to defend its own output when it's also the judge. A grader who wrote the exam is a suspect grader.

The way to kill this is structural. The reviewer runs as a separate agent, with its own context, sometimes on a different model, with the single mission of trying to [knock the result down before it gets accepted](/posts/2026/05/25/code-review-novo-gargalo-coderabbit/). In the workflow you put one adversarial verifier per output. In the end, the agents themselves open the PRs. Whoever produces is NEVER whoever approves.

## It's expensive, and the ROI is niche

I'll be honest, because the part nobody posts is the cost. Dynamic Workflows is a research preview and it burns tokens without mercy. There are reports of people torching the five-hour limit in eighteen minutes, and of runs of three million tokens without a single cost warning along the way. This isn't free scale.

So who does this pay off for?

For whoever has the seniority to review. The senior's leverage is judgment: knowing when the AI spat out slop, correcting course, blocking the bad task. A junior on the same tool is money down the drain, because without real software engineering they accept whatever comes and bang their head against the final result. The ROI is glued to seniority, not to the tool.

This becomes the default the day running thirty tasks in parallel, each with its own reviewer, costs the same as running one by hand. Whoever wants to anticipate that day already makes the token hurt less with model routing: most of the tasks on a cheap model, the expensive one only on the plan and the review.

{{< cta servico="discovery" variante="mid" />}}

## The tool changes, the physics is the same

The most interesting part isn't any specific tool. It's that everyone, starting from different places, is arriving at the same two rules.

**Rule one: the project's memory lives in the files, not in the context.** [ADR in the repo](/posts/2026/05/16/adrs-decisao-no-notion-sem-burocracia/), `project-context.md`, `state.json`, `todo.md`, a versioned decision matrix. The agent doesn't need to "remember" the decision from task four. It reads the file. The context rot disappears because you stopped stacking history in the window.

**Rule two: the reviewer is never the author, by construction.** Separate contexts for whoever generates and whoever validates. The validator walks in assuming there's a bug and goes hunting.

Look at how many people arrived at this from opposite paths:

- **Ralph loop** (Geoffrey Huntley): wraps the agent in a `while`, clean context on every turn, memory on disk. Monolithic on purpose. He rejects multi-agent, and even so externalizes state in exactly the same way.
- **Dynamic Workflows** (Anthropic): the opposite of Ralph, fan-out of hundreds of agents, but the script holds the state and the adversarial reviewer is separate.
- **BMAD, MDDD, cstk**: community frameworks that, each in its own way (ADR plus adversarial reviewer, decision matrix, waves with `state.json` and model routing), implement the same two rules.

> "You're just reinventing a `while` loop with more steps."

In part, yes. The Ralph loop is the rawest form of this, and it works. The difference is what you hang on top: consolidator, separate reviewer, model routing, [all coded into a harness](/posts/2026/05/16/claude-code-superpowers-plugin-na-pratica/) instead of in your three-in-the-morning prompt. The principle is old. The discipline of applying it is what changes the result.

## The work you thought was thinking

Spec-Driven Development didn't fail. It solved the part you could solve by writing, and exposed the part that was missing: executing without the context rotting and without you in the middle of the loop copying output from one side to the other.

The way out isn't a tool. It's a physics: state outside the window, reviewer outside the author. Dynamic Workflows, Ralph loop, cstk, BMAD, they're accents of the same sentence.

The work you thought was thinking was always managing context. AI didn't change that. It just made it obvious.
