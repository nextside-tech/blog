---
title: "Playwright"
description: "Playwright em CI bloqueia regressão. Playwright via MCP deixa o Claude dirigir o browser, tirar screenshot e flagrar bug visual antes do PR."
---

Playwright é a stack de automação de browser do Microsoft. Headless ou não. Cross-browser (Chromium, Firefox, WebKit). API consistente, performante, com excelente DX. O que Selenium queria ser e nunca conseguiu.

Tem dois jeitos de usar hoje, e eles não competem. O Playwright tradicional é spec em código, versionado no repo, rodando em CI a cada PR — esse é o suite E2E que bloqueia merge quando regressão real aparece. O Playwright via MCP é diferente: Claude ganha olhos no browser, navega seu app local, tira screenshot, lê o DOM, verifica console. Sem você escrever spec de teste.

Como exploramos em [MCP Playwright: validação local com qualidade real](/posts/2026/05/16/mcp-playwright-validacao-local-com-qualidade/), a IA cansa menos que você. Não esquece de testar dark mode. Não pula mobile na pressa. Não diz "ah, depois eu vejo o console". Toda vez que roda, roda tudo.

## Por que este tópico importa

O ângulo Nextside: Playwright via MCP não substitui CI. ABSOLUTAMENTE NÃO. Complementa. É o passo 3 do fluxo de dev — aquele clique manual de pré-PR que todo mundo pula — automatizado pelo agent. O suite E2E em CI continua sendo o seguro que bloqueia regressão crítica. MCP Playwright é o pré-voo que pega o bug visual antes do PR sair.

Quem trocar suite E2E por MCP Playwright vai sentir saudade no primeiro refactor grande. Quem usar os dois ganha consistência que time só com CI nunca alcança com força de vontade humana.

## Posts marcados
