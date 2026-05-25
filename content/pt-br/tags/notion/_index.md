---
title: "Notion"
description: "Notion como base de conhecimento operacional: ADRs, especificações, histórico de decisão técnica — em um lugar que humano e IA leem juntos."
---

Notion não é "wiki bonita". É database relacional com markdown decente em cima. E é por isso que funciona melhor que Confluence, Word ou markdown solto no Drive pra time pequeno tocando produto com IA pesada.

Confluence é caro, lento e tem busca ruim. Word vive perdido em pasta compartilhada. Markdown solto no Drive depende de cada um lembrar de versionar. Notion entrega database de verdade — schema, filtro, view, API — sem precisar de ops de tooling. Cria uma base `Decisões`, define 5 campos, e a memória institucional já está num lugar consultável.

O detalhe que muda o jogo em 2026: Notion tem API estável e tem MCP server pronto. Isso significa que o Claude Code consulta sua base de ADRs antes de propor decisão técnica nova — sem você abrir o navegador, sem copiar e colar contexto, sem mandar a IA "lembrar" do que já foi decidido.

## Por que este tópico importa

O ângulo Nextside: documentação clássica falha porque ninguém atualiza e ninguém lê. ADR no Notion não falha do mesmo jeito porque o critério de gatilho é pequeno (só decisões caras de reverter) e a IA também passa a consumir a base. Quando o documento vira input pro próximo prompt automaticamente, ele para de ser cemitério.

Database de ADRs é o exemplo mais claro. Mas o mesmo princípio vale pra especs, runbooks, histórico de decisão de produto: o lugar onde humano e IA leem juntos é o lugar que sobrevive ao cansaço do time.

## Posts marcados
