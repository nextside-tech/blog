---
title: "Spec-driven development: sair do vibe coding travado"
date: 2026-06-12T09:00:00-03:00
draft: false
translationKey: spec-driven-development-sair-do-vibe-coding
category: tecnologia
tags: [spec-driven-development, vibe-coding, ai-tooling, agentes-de-ia, arquitetura]
author: lucas-israel
description: "Por que o vibe coding empaca na hora de escalar — e como o spec-driven development entrega uma ordem de grandeza menos retrabalho com agentes de IA."
cover: covers/spec-driven-development-sair-do-vibe-coding.png
cover_alt: "Especificação como artefato central de um fluxo de IA: o agente lê a spec versionada e gera código a partir dela, em vez de adivinhar a partir de um prompt solto"
cta:
  servico: discovery
  posicao: ambos
  contexto: "Tem uma ideia travada ou um MVP vibe-coded que precisa virar produto? Um Discovery transforma isso em spec executável — roadmap + protótipo em 2-3 semanas."
canonical: ""
---

## TL;DR

Você prototipou rápido com IA. Agora o app não escala, e cada feature nova quebra duas antigas. Esse é o ponto exato onde o vibe coding para de ajudar — e onde o <dfn>spec-driven development (SDD)</dfn> começa a pagar. A ideia é simples e inverte a ordem do jogo: a **especificação vira o artefato principal**, e o agente implementa a partir dela em vez de adivinhar. O trade-off é real — você troca a euforia do "deu certo de primeira" por 30 minutos escrevendo spec antes de codar. Pra protótipo descartável, não compensa. Pra o que vai pra produção e precisa crescer, é o que separa entrega de gambiarra.

Vibe coding é ótimo pra descobrir o quê construir. É péssimo pra sustentar o que já existe.

## O vibe coding não falha por ser IA. Falha por ser ambíguo.

Num prompt solto, o modelo tem 30 formas de implementar a mesma feature — e roda a mesma instrução duas vezes, sai diferente. Essa ambiguidade é tolerável no protótipo e fatal na manutenção: ninguém — nem você, nem o próximo dev, nem o agente — sabe qual era a regra. O código é a única fonte de verdade, e ela muda a cada geração.

Se você é CTO e ainda está no vibe coding, o sintoma é familiar: o MVP saiu numa semana, o time dobrou a velocidade no começo, e agora cada PR de IA precisa de três rodadas de review porque o agente "esqueceu" uma decisão que nunca foi escrita em lugar nenhum. Vi isso de perto mais de uma vez — o pessoal acha que é questão de contratar mais um sênior. Não é.

**O gargalo deixou de ser escrever código. Virou alinhar contexto.**

## O que muda no spec-driven development

SDD coloca a especificação **antes** da geração de código: requisitos, regras de negócio, contratos de API e restrições de arquitetura viram um documento que o agente lê e segue. A spec é versionada, revisada e reusada — o código passa a ser saída, não a fonte de verdade. Menos adivinhação, menos loop de "não era isso".

Na prática o fluxo é direto: você descreve o comportamento e as restrições → o agente propõe um plano contra a spec → você valida o plano (não 400 linhas de diff) → o agente implementa e testa contra os critérios que a própria spec definiu. O review deixa de ser "isso está certo?" e vira "isso bate com a spec?" — uma pergunta que dá pra responder em minutos.

Não é teoria de blog. Em projetos internos com o [Spec Kit, o GitHub relata cerca de uma ordem de grandeza menos ciclos de "regerar do zero"](https://www.turingpost.com/p/sdd) que prompting ad-hoc. A AWS documenta com o Kiro [casos de features de 40 horas entregues em menos de 8 horas de tempo humano](https://towardsdatascience.com/from-vibe-coding-to-spec-driven-development/) quando o trabalho começou pela spec. E o próprio criador do termo "vibe coding", Andrej Karpathy, já reconheceu publicamente o limite da abordagem pra software de verdade.

### SDD funciona com agentes de IA como Claude e Copilot?

Sim, e é exatamente pra isso que foi feito. Ferramentas como GitHub Spec Kit e AWS Kiro integram com agentes como Claude Code, Copilot e Gemini CLI. A spec vira o contexto que o agente segue — o mesmo papel que um `CLAUDE.md` bem escrito cumpre no dia a dia, só que elevado a artefato de primeira classe do projeto.

{{< cta servico="discovery" variante="mid" />}}

## Onde isso quebra

SDD não é bala de prata, e fingir que é seria cair no mesmo erro do hype do vibe coding.

> "Isso é só mais cerimônia. Mais um documento bonito que ninguém vai ler."

Pode ser. Esse é o risco real, e eu já vi virar isso. Escrever spec custa tempo de cabeça — pra um spike de um dia, um teste de hipótese ou um throwaway, o overhead não se paga: vibe coding ganha. SDD também é tão bom quanto a spec: spec vaga gera código vago, e você só transferiu a ambiguidade pra um documento mais bonito.

A régua que uso é simples: **se o código vai sobreviver mais de um mês ou passar pela mão de outra pessoa, especifique. Se é pra jogar fora, não.** A decisão é por estágio, não por dogma.

### Spec-driven development substitui o vibe coding?

Não pra tudo. Vibe coding continua ótimo pra protótipos, spikes e validação de hipótese — onde a velocidade de descobrir vale mais que a disciplina de sustentar. SDD ganha quando o código vai pra produção, precisa escalar ou passar pela mão de outras pessoas. Não é um substituindo o outro. É saber em que estágio você está.

## Como sair do vibe coding sem parar o time

Não precisa reescrever tudo. A migração é incremental e começa na próxima feature, não num big bang:

- **Escreva a spec antes de chamar o agente** — mesmo que curta. Comportamento esperado, regras e restrições. Cinco linhas já mudam o jogo.
- **Toda regra de negócio mora na spec** — não num comentário, não no Slack, não na cabeça de alguém. Se não está na spec, não existe pro agente.
- **Use a spec como critério de review** — a pergunta deixa de ser "isso está bom?" e vira "isso bate com o que a gente especificou?".

Em poucas semanas o retrabalho cai, porque o contexto parou de evaporar entre uma geração e outra.

### SDD deixa o desenvolvimento mais lento?

No começo de cada feature, sim — você investe os tais 30 minutos escrevendo a spec. No total, costuma ser mais rápido: é a diferença entre a ordem de grandeza menos ciclos de regerar do zero que o GitHub relata e as features de 40h entregues em menos de 8h que a AWS reporta. Você paga adiantado pra não pagar o juro composto do retrabalho depois.

## A spec é o contexto que não evapora

Vibe coding te dá o primeiro quilômetro de graça e cobra o resto da estrada em retrabalho. SDD faz o oposto: cobra adiantado e devolve previsibilidade.

O ponto não é abandonar a IA — é parar de tratar o código gerado como fonte de verdade. A fonte de verdade é a spec. O código é só a saída.

Se a sua equipe vai gastar IA de qualquer jeito, gaste no que está especificado.
