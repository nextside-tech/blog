---
title: "ADR (Architecture Decision Record)"
description: "ADR sem burocracia: registrar decisão técnica em 3 campos no Notion, datado, curto, lido por humano e por IA. Sem template corporativo de 14 páginas."
---

ADR é registro de decisão técnica. Curto. Datado. Imutável. Você decide algo importante hoje, escreve por que decidiu, e daqui a 6 meses a resposta tá lá — sem precisar reconvocar a reunião perdida no calendário de fevereiro.

O ponto não é o template. O ponto é não perder história. A maioria dos times copia o template clássico do Michael Nygard com 6 seções, escreve 3 ADRs em 4 dias, e abandona no quinto. Time pequeno não tem paciência pra ritual de auditor. Por isso o formato Nextside é menor: título, status, data, três parágrafos de body. Acabou.

Documentar tudo é o mesmo que documentar nada — vira ruído. A regra de gatilho é objetiva: se a decisão custa 1+ dia pra reverter, escreve ADR. Se cabe num PR de 50 linhas, não escreve nada.

## Por que este tópico importa

ADR mudou de patamar quando a IA entrou no fluxo. Antes, o documento servia ao humano novo que entrava no projeto seis meses depois. Agora também serve ao Claude Code que abre o repo do zero a cada sessão — e que consulta o INDEX de ADRs antes de propor qualquer decisão arquitetural.

Como mostramos em [ADRs no Notion, sem burocracia](/posts/2026/05/16/adrs-decisao-no-notion-sem-burocracia/), o valor real do ADR não está no documento. Está no ato de escrever — que força clareza sobre uma decisão que estava implícita e mal formada. O INDEX leve, lido pela IA, garante que essa clareza chega na próxima implementação sem alguém ter que lembrar.

## Posts marcados
