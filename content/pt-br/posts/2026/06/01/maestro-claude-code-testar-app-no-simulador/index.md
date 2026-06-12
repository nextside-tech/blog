---
title: "Maestro + Claude Code: seu app testado no simulador como o Playwright testa a web"
date: 2026-06-01T09:00:00-03:00
draft: false
translationKey: maestro-claude-code-testar-app-no-simulador
category: tecnologia
tags: [maestro, mobile-testing, react-native, claude-code, qa-automation]
author: bruno-raphael
description: "Dá pra fazer o Claude navegar e testar seu app no simulador como o Playwright faz na web. Com Maestro: um YAML, iOS e Android, zero instrumentação."
cover: covers/maestro-claude-code-testar-app-no-simulador.png
cover_alt: "Capa do Blog Nextside com o título 'Seu app testado no simulador' e o símbolo N da Nextside em vermelho sobre fundo escuro"
cta:
  servico: discovery
  posicao: ambos
  contexto: "Quer botar o Claude pra testar seu app mas não sabe se cola no seu stack? A gente valida o setup com você num Discovery."
canonical: ""
---

## TL;DR

O Claude Code já navega seu site sozinho via Playwright: clica, preenche, valida regressão. Pra app mobile dá pra fazer a mesma coisa, mas ninguém explica direito como. Fui atrás. A resposta é o <dfn>Maestro</dfn>, um framework open source de teste E2E mobile com flows escritos em YAML, plugado no Claude Code. Um único arquivo de teste roda igual em iOS e Android, sobre o binário compilado, sem instrumentar o app. React Native, nativo ou Flutter, tanto faz. O Claude inspeciona a tela, escreve o flow, roda e conserta o que quebra. E não: o caminho certo NÃO é "dar acesso à tela pro Claude". Screenshot por coordenada é o último recurso, não o primeiro.

Esse post é o setup que montei pra fechar no mobile o buraco que o [Playwright já fechou pra web](/posts/2026/05/16/mcp-playwright-validacao-local-com-qualidade/). [Aqui na Nextside](https://nextside.tech) ainda não virou pipeline de produção. É o caminho que estou adotando, com a engenharia destrinchada, os comandos na mão e os números de quem já trilhou ele.

## Não é "dar acesso à tela". É ler a árvore.

Quando eu conto isso, vem sempre a mesma pergunta, e eu também fiz ela no começo: "o Claude não consegue só olhar a tela e clicar, igual um humano?". Consegue. Chama <dfn>Computer Use</dfn>: o Claude controla a interface por screenshot e clique em coordenada de pixel. Lançou no Claude Code em março de 2026, dirige o simulador, funciona pra demo.

Só que é o jeito errado pra teste.

O Playwright que você já usa nunca olhou pixel nenhum. Ele lê o *accessibility tree*, a árvore estruturada que descreve "botão rotulado Entrar, aqui". Age por elemento, não por coordenada. É por isso que é rápido e não alucina onde clicar.

A diferença dá pra medir em token: a árvore de acessibilidade de uma tela sai por [uns 10 tokens, e um screenshot da mesma tela custa de 1.600 a 6.300](https://github.com/conorluddy/ios-simulator-skill). Multiplica por cada passo de um teste de vinte telas e você entende por que visão não escala num loop de QA.

No fundo são três jeitos de fazer o Claude mexer no app, do melhor pro pior:

- **MCP ou CLI lendo a árvore.** Estruturado, determinístico, barato em token. É o "jeito Playwright", e é onde o Maestro vive.
- **Computer Use por screenshot.** O Claude enxerga a tela e chuta coordenada. Generaliza pra qualquer app, mas é lento (2 a 5 segundos por ação), erra clique e queima contexto.
- **Nada.** Você testando tudo na mão, que é de onde a gente tá saindo.

A própria Anthropic ordena assim. A [hierarquia de ferramentas do Claude Code](https://code.claude.com/docs/en/computer-use) é MCP primeiro, depois shell, depois Chrome, e só cai pro controle de tela quando nada mais alcança: "apps nativos, simuladores e ferramentas sem API".

**Screenshot é o último recurso, não o primeiro.**

## Maestro: um YAML, iOS e Android, zero instrumentação

Se o jeito certo é ler a árvore, eu preciso de uma ferramenta que exponha a árvore do simulador pro Claude. Tem várias. Eu fechei no Maestro, e pra quem mantém React Native e app nativo, ele ganha por três motivos concretos:

- **Opera na camada de acessibilidade, sobre o binário compilado.** Não importa se o app é React Native, Swift/Kotlin nativo ou Flutter. O Maestro testa o APK/IPA pronto, sem driver instalado nem mudança no código-fonte. Pra um time que toca RN e nativo lado a lado, isso é o fim de manter duas stacks de teste.
- **O mesmo arquivo roda nos dois sistemas.** Você escreve o flow uma vez. Roda no simulador do iPhone e no emulador do Android sem reescrever uma linha.
- **YAML que humano e máquina leem.** Não é código com seletor frágil. É uma sequência declarativa que o Claude gera e edita na hora.

Um flow do Maestro começa simples assim:

```yaml
appId: com.suaempresa.app
---
- launchApp
- tapOn: { id: "login_button" }
- inputText: "user@nextside.tech"
- tapOn: "Entrar"
- assertVisible: "Bem-vindo"
```

`appId`, três hífens, e os comandos em linguagem quase natural: `launchApp`, `tapOn`, `inputText`, `assertVisible`. Quem nunca viu entende em dez segundos.

Onde isso fica sério é no reuso. O login se repete em todo teste, então você extrai ele uma vez e chama com `runFlow`:

```yaml
# flows/login.yaml
appId: com.suaempresa.app
---
- launchApp: { clearState: true }
- tapOn: { id: "login_button" }
- inputText: "user@nextside.tech"
- tapOn: "Entrar"
```

```yaml
# flows/comprar.yaml
appId: com.suaempresa.app
---
- runFlow: login.yaml          # reusa o login inteiro
- tapOn: { id: "produto_42" }
- scrollUntilVisible:
    element: { text: "Finalizar compra" }
- tapOn: "Finalizar compra"
- assertVisible: "Pedido confirmado"
```

Muda a regra de login num lugar, vale nos vinte testes que chamam ele. Repara no `scrollUntilVisible` e no `clearState: true`: o Maestro tem comando pra rolar até achar, limpar estado, trocar permissão, setar localização. E **espera o elemento aparecer sozinho**, sem você espalhar `sleep` pelo teste. Sleep é cheiro de teste mal feito, aqui não precisa.

**Mesmo arquivo. iOS e Android. Sem tocar no código do app.**

## Do zero ao primeiro teste

O "como usar" de verdade começa antes do Claude. Você precisa de três coisas na máquina:

- **Java 17 ou mais novo.** O motor do Maestro roda em JVM. Confere com `java -version`.
- **Xcode e o Command Line Tools.** É o que destrava o simulador iOS.
- **Android platform-tools com o `$ANDROID_HOME` setado e um emulador rodando.** Confere com `adb devices`.

Com isso no lugar, instala o Maestro num comando:

```bash
curl -fsSL "https://get.maestro.mobile.dev" | bash
# ou, no macOS, via Homebrew:
# brew install mobile-dev-inc/tap/maestro
maestro --help   # confirma que tá vivo
```

Boota um simulador (ou emulador), instala seu app nele, e roda o flow:

```bash
maestro test flows/comprar.yaml      # um flow
maestro test flows/                  # a pasta inteira
```

Só isso já te dá teste E2E rodando local, sem IA nenhuma. A IA entra pra você parar de escrever esses YAMLs na mão.

## O loop na prática: o Claude escreve o teste olhando o app

Conecta o Maestro ao Claude Code num comando:

```bash
claude mcp add maestro -- maestro mcp
```

Isso entrega ao Claude um punhado de ferramentas: `inspect_screen` (pega a view hierarchy da tela como JSON compacto), `run` (executa um flow) e `open_maestro_viewer` (embute o simulador numa janela onde você vê cada comando rodar em tempo real).

O loop que isso destrava muda o jogo:

1. **Claude inspeciona** a tela ao vivo. Lê a árvore, não adivinha.
2. **Claude escreve** o flow YAML, [sem você caçar element ID na mão](https://maestro.dev/blog/maestro-mcp-an-introduction).
3. **Claude roda** no simulador.
4. **Claude diagnostica** o que falhou olhando a hierarquia, e conserta o próprio teste.

O passo 4 é o que mais economiza saúde. Quando um `tapOn: "Entrar"` quebra porque o botão virou "Acessar" numa refatoração, o fluxo manual é: teste falha no CI, alguém abre, descobre, corrige o seletor, sobe de novo. Com o loop, o Claude relê a hierarquia, vê que o rótulo mudou, troca pro `id` estável e te mostra o diff. Você aprova ou não. [O Maestro chama isso de self-healing](https://maestro.dev/blog/maestro-mcp-an-introduction). É a manutenção de teste, a parte mais chata de QA, saindo das suas costas.

No React Native, o que faz esse loop ser confiável é o `testID`. O que você já põe nos componentes vira o `id` do Maestro direto:

```jsx
<Button title="Entrar" testID="login_button" onPress={onLogin} />
```

Prefira `testID` a texto sempre. Texto muda com tradução e com revisão de copy. O `testID` só muda se você mexer nele de propósito. E quando não souber qual seletor existe numa tela, `maestro studio` abre um inspetor visual no browser: você clica no elemento, ele mostra os seletores disponíveis e gera o YAML do passo. É assim que você ensina o Claude a mirar nos lugares certos do seu app.

### MCP ou Skill+CLI: qual usar?

Os dois funcionam. A escolha é sobre contexto. O **MCP** é plug-and-play: um comando e o Claude tem as ferramentas. O preço é que todo MCP carrega o schema das tools no contexto do modelo, e isso come token a cada sessão.

A alternativa é uma **Skill** que ensina o Claude a rodar `maestro test flow.yaml` direto no terminal. Mais enxuto, porque você não paga o overhead do servidor. A [própria comunidade está migrando de MCP pra Skill+CLI por isso](https://news.ycombinator.com/item?id=45642911). Minha regra: começo no MCP pra explorar e prototipar rápido. Quando o fluxo vira rotina, encapsulo numa Skill com o CLI e largo o servidor.

{{< cta servico="discovery" variante="mid" />}}

## O pedágio do iOS (a parte que ninguém posta)

Agora a parte honesta, porque vender isso como mágica é desserviço.

Primeiro: teste gerado por IA acerta [70 a 80% na primeira passada](https://medium.com/coding-nexus/someone-built-a-tool-that-lets-claude-code-autonomously-test-your-entire-ios-app-5256d0a49703). O Claude escolhe o seletor errado, esquece um wait. O fluxo que presta é deixar a IA gerar a v1, rodar uma vez pra validar, e devolver a manutenção pra ela. Não é "manda e esquece".

Segundo, e pesado pra quem é de mobile: **o iOS cobra pedágio.** Um dev documentou montar o mesmo QA nas duas plataformas. [Android levou 90 minutos, o iOS passou de seis horas](https://christophermeiklejohn.com/ai/zabriskie/development/android/ios/2026/03/22/teaching-claude-to-qa-a-mobile-app.html). A frase dele resume a década inteira de automação mobile. "Android te dá um WebSocket e diz: aqui está o app, faça o que quiser. iOS te dá uma porta trancada e um bilhete pedindo pra usar o Xcode."

A boa notícia é que o Maestro abstrai boa parte desse pedágio, é o mesmo `tapOn` nos dois. Mas duas pedras você ainda vai pisar no React Native:

- **Componente aninhado no iOS.** O iOS "engole" o toque quando você tem um `Text` dentro de um `TouchableOpacity` dentro de outro container tocável. A correção é `accessible={false}` no container de fora e `accessible={true}` no elemento de dentro. É chato, mas é uma vez por componente.
- **Expo Go não aceita `launchApp`.** Rodando via Expo Go, o app vive dentro do container do Expo, e o `launchApp` com seu `appId` não pega. Tem que usar `openLink` com a URL de dev, ou fazer um development build de verdade (EAS). Em bare React Native, `launchApp` funciona normal.

> "Vocês vão deixar um bot escrever e rodar os testes do app? Isso vai dar ruim."

Vai dar ruim se você tratar o teste gerado como verdade e largar. Não vai se você tratar como rascunho que o sênior revisa, igual você já faz (ou devia fazer) com código que a IA escreve. O Maestro ainda te entrega o YAML versionado: dá pra ler no PR, discordar, corrigir. O teste continua sendo seu. O Claude só parou de te fazer digitar ele do zero.

## De teste solto a rotina

Um teste que você roda na mão quando lembra não é rede de segurança. É teatro. O ganho real aparece quando o flow vira rotina automática. Como o Maestro é só um binário de linha de comando, ele entra em qualquer lugar que rode shell:

```bash
maestro test flows/    # roda a suíte inteira; sai com código de erro se quebrar
```

Esse `maestro test flows/` é a mesma linha que você roda local, no GitHub Actions a cada PR, ou num cron noturno. Aquele dev do case real [deixou a suíte rodando como tarefa agendada toda manhã às 8:47](https://christophermeiklejohn.com/ai/zabriskie/development/android/ios/2026/03/22/teaching-claude-to-qa-a-mobile-app.html): boota os dois simuladores, varre as telas, analisa, e abre report do que parece quebrado. O dev acorda com o QA já feito.

O ciclo fecha aqui. O Claude escreve o flow olhando o app, o flow vira arquivo versionado, o arquivo roda no CI. A IA monta a rede, a máquina puxa ela toda noite.

## A IA escreve o código e o teste. Você ainda decide o que é "funciona".

A gente [já falou aqui que a IA revisa código mas não testa software](/posts/2026/05/25/code-review-novo-gargalo-coderabbit/). Continua verdade, com um asterisco novo: agora ela TESTA, no simulador, navegando o app como usuário faria. O que ela não faz é decidir o que conta como "funcionou".

Esse julgamento é seu. O critério de aceite é seu. O Maestro e o Claude tiram de você a parte chata: bootar o simulador, caçar o ID do botão, digitar o flow, rodar nos dois sistemas, consertar o seletor que mudou. Devolvem o tempo pra única coisa que a máquina não faz: olhar o app e decidir se está bom.

Ferramenta boa não substitui critério. Ela só tira a desculpa de não ter testado.
