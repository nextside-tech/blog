---
title: "MCP (Model Context Protocol)"
description: "MCP — Model Context Protocol: protocolo aberto da Anthropic que liga LLMs a ferramentas externas como GitHub, Notion ou Playwright. USB pra IA."
---

MCP é o Model Context Protocol — um protocolo aberto criado pela Anthropic pra ligar LLMs a ferramentas externas. Pensa em USB pra IA: padrão único, plug, e qualquer cliente compatível (Claude Code, Claude Desktop, Cursor) conversa com qualquer MCP server do mercado.

Antes de MCP, integrar IA com ferramenta externa era artesanal. Cada cliente tinha sua forma de invocar tools. Cada tool exigia adaptador específico. Caos. O protocolo resolve isso com três pedaços simples: cliente (onde a IA roda), servidor MCP (processo que expõe ferramentas), e tools (o que o servidor oferece — "navegue pra URL", "leia este arquivo", "execute essa query").

Hoje tem MCP server pra praticamente tudo: GitHub, Linear, Notion, Postgres, browser via Playwright, filesystem, Slack. Você pluga o que precisa, e a IA passa a operar essas ferramentas como extensão do próprio cliente.

## Por que este tópico importa

O foco aqui é no que dá pra usar agora, não no hype. MCP Playwright valida frontend local antes do PR. MCP Notion consulta base de conhecimento sem você abrir o navegador. MCP Postgres deixa o agent rodar query de diagnóstico no banco de dev. Tudo padronizado, tudo plugável.

O ângulo Nextside: a graça do MCP não é cada server individual — é parar de reinventar integração artesanal. Quando o protocolo é padrão, o esforço vai pra resolver o problema real, não pra fazer a IA falar com a ferramenta certa.

## Posts marcados
