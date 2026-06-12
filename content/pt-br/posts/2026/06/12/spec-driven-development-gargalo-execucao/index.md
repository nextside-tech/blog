---
title: "A spec era a parte fácil. O gargalo do SDD é a execução"
date: 2026-06-12T09:00:00-03:00
draft: false
translationKey: spec-driven-development-gargalo-execucao
category: tecnologia
tags: [spec-driven-development, dynamic-workflows, claude-code, context-rot, orquestracao, ia-development]
author: pablo-winter
description: "Spec-Driven Development externaliza a intenção em markdown, mas empurra a dor pra execução. Por que orquestrar com o estado fora da janela do modelo é a saída."
cover: covers/spec-driven-development-gargalo-execucao.png
cover_alt: "Capa editorial escura Nextside: o gargalo de execução do Spec-Driven Development e uma spec rodando por agentes de IA"
cta:
  servico: discovery
  posicao: ambos
  contexto: "Vai botar agente pra executar spec no seu time? A gente valida onde a IA acelera de verdade num Discovery, antes de você queimar budget."
canonical: ""
---

## TL;DR

Spec-Driven Development resolveu um problema real: você externaliza a intenção em markdown versionado (PRD, tech spec, lista de tasks) e a spec vira a fonte da verdade. Só que ninguém te conta a conta da execução. Uma spec gera dezenas de tasks, e rodar tudo numa conversa só é onde o contexto degrada e você vira gerente de janela. A saída que a indústria inteira convergiu, de framework caro a `while` loop de bash, é a mesma: tirar o estado da janela do modelo e botar em arquivo ou código, com um revisor que nunca é quem escreveu. Os <dfn>Dynamic Workflows</dfn> da Anthropic — onde o próprio Claude escreve o script que orquestra os agentes — são uma forma disso. Tem várias.

Esse post é sobre por que a execução era o gargalo o tempo todo, e por que todo mundo está chegando nas mesmas duas regras.

## Ninguém te conta a conta da execução

Spec-Driven Development é simples de descrever: você escreve a intenção antes do código. PRD vira tech spec, tech spec vira lista de tasks atômicas, e só então o agente gera código. GitHub Spec Kit, Amazon Kiro, Tessl, cada um com seu sabor. A spec é a fonte da verdade, o código é consequência.

Escrever a spec é a parte fácil.

Minha última spec gerou trinta e poucas tasks. O inferno não começou ali. Começou na hora de executar as trinta e poucas numa conversa só. Você roda task atrás de task, a janela enche, e lá pela vigésima o agente já esqueceu a decisão que ele mesmo tomou na quarta.

Isso tem nome e foi medido. <dfn>Context rot</dfn> — a queda de qualidade do modelo conforme o contexto cresce — foi testado pela Chroma em 18 modelos. Os 18 degradaram, e a degradação começa bem antes da janela encher. O paper "Lost in the Middle" já tinha mostrado a mesma curva: o modelo perde a informação enterrada no meio de um contexto longo.

O remendo que a comunidade adotou é abrir uma janela limpa por task: contexto novo, recola a spec, aponta a task, executa, repete. Funciona contra o rot. E te transforma em estagiário de copy-paste, trinta e poucas vezes.

**A spec era a parte fácil.**

## As três fases de quem carrega o contexto

O gargalo sempre foi o mesmo: alguém precisa segurar o estado e consolidar os resultados enquanto as tasks rodam. O que mudou foi quem carrega esse peso.

**Fase 1, na mão.** Você é a janela de contexto. Roda task por task, dá `/clear`, relê a spec, segura o estado na cabeça e na conversa. Vai bem pra cinco tasks. Na trigésima, você já é o gargalo.

**Fase 2, delegando.** Você joga a execução pros subagentes. Ajuda. Só que o output de todos volta pra mesma janela, a do agente principal que você está dirigindo, e é essa janela que vira o consolidador e apodrece. Agent Teams melhoraram com uma task list compartilhada, mas o lead ainda dirige passo a passo. O gargalo mudou de lugar, não sumiu.

**Fase 3, workflow.** Aqui muda a física. O plano sai do seu contexto e vira código. Um script segura o loop e os resultados intermediários, e o contexto do modelo só vê a resposta final. Cada task roda numa janela isolada. Foi aqui que eu finalmente parei de ser o gargalo. É o que os Dynamic Workflows do Claude Code fazem: o próprio Claude escreve um script JavaScript de orquestração, e um runtime executa em segundo plano, com até 16 agentes simultâneos e teto de mil por run.

Jarred Sumner, criador do Bun, levou isso ao extremo. Portou o Bun de Zig pra Rust exatamente nesse esquema: tasks em paralelo, dois revisores contestando cada arquivo. Setecentas e cinquenta mil linhas de Rust, 99,8% da suíte de testes passando, onze dias do primeiro commit ao merge. Ainda não foi pra produção, é demonstração de capacidade. Mas o número é esse.

### Por que o revisor não pode ser quem escreveu?

Porque o modelo tem viés de auto-preferência. Self-preferential bias é a tendência do modelo de defender o próprio output quando ele mesmo é o juiz. Um corretor que escreveu a prova é um corretor suspeito.

O jeito de matar isso é estrutural. O revisor roda como um agente separado, com contexto próprio, às vezes num modelo diferente, com a única missão de tentar [derrubar o resultado antes dele ser aceito](/posts/2026/05/25/code-review-novo-gargalo-coderabbit/). No workflow você bota um verificador adversarial por output. No fim, os próprios agentes abrem os PRs. Quem produz NUNCA é quem aprova.

## Custa caro, e o ROI é de nicho

Vou ser honesto, porque a parte que ninguém posta é o custo. Dynamic Workflows é research preview e queima token sem dó. Tem relato de gente torrando o limite de cinco horas em dezoito minutos, e de runs de três milhões de tokens sem um aviso de custo no meio. Não é escala de graça.

Então pra quem isso paga?

Pra quem tem senioridade pra revisar. A alavanca do sênior é julgamento: saber quando a IA cuspiu slop, corrigir rota, barrar a task ruim. Júnior na mesma ferramenta é dinheiro no ralo, porque sem engenharia de software de verdade ele aceita o que vier e bate cabeça no resultado final. O ROI anda colado na senioridade, não na ferramenta.

Isso vira default no dia que rodar trinta tasks em paralelo, cada uma com seu revisor, custar o mesmo que rodar uma na mão. Quem quer antecipar esse dia já faz o token doer menos com roteamento de modelo: a maior parte das tasks num modelo barato, o caro só no plano e no review.

{{< cta servico="discovery" variante="mid" />}}

## Ferramenta muda, a física é a mesma

A parte mais interessante não é nenhuma ferramenta específica. É que todo mundo, partindo de lugares diferentes, está chegando nas mesmas duas regras.

**Regra um: a memória do projeto vive nos arquivos, não no contexto.** [ADR no repo](/posts/2026/05/16/adrs-decisao-no-notion-sem-burocracia/), `project-context.md`, `state.json`, `todo.md`, matriz de decisão versionada. O agente não precisa "lembrar" da decisão da task quatro. Ele lê o arquivo. O context rot some porque você parou de empilhar histórico na janela.

**Regra dois: o revisor nunca é o autor, por construção.** Contextos separados pra quem gera e pra quem valida. O validador entra assumindo que tem bug e vai caçar.

Olha o tanto de gente que chegou nisso por caminhos opostos:

- **Ralph loop** (Geoffrey Huntley): embrulha o agente num `while`, contexto limpo a cada volta, memória no disco. Monolítico de propósito. Ele rejeita multi-agente, e ainda assim externaliza o estado igualzinho.
- **Dynamic Workflows** (Anthropic): o oposto do Ralph, fan-out de centenas de agentes, mas o script segura o estado e o revisor adversarial é separado.
- **BMAD, MDDD, cstk**: frameworks da comunidade que, cada um do seu jeito (ADR mais reviewer adversarial, matriz de decisão, ondas com `state.json` e roteamento de modelo), implementam as mesmas duas regras.

> "Vocês tão só reinventando um `while` loop com mais etapas."

Em parte, sim. O Ralph loop é a forma mais crua disso, e funciona. A diferença é o que você pendura em cima: consolidador, revisor separado, roteamento de modelo, [tudo codificado num harness](/posts/2026/05/16/claude-code-superpowers-plugin-na-pratica/) em vez de no seu prompt de três da manhã. O princípio é velho. A disciplina de aplicá-lo é o que muda o resultado.

## O trabalho que você achava que era pensar

Spec-Driven Development não falhou. Ele resolveu a parte que dava pra resolver escrevendo, e expôs a parte que faltava: executar sem o contexto apodrecer e sem você no meio do circuito copiando output de um lado pro outro.

A saída não é uma ferramenta. É uma física: estado fora da janela, revisor fora do autor. Dynamic Workflows, Ralph loop, cstk, BMAD, são sotaques da mesma frase.

O trabalho que você achava que era pensar sempre foi gerenciar contexto. A IA não mudou isso. Só deixou na cara.
