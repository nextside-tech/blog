---
title: "Maestro + Claude Code: your app tested in the simulator like Playwright tests the web"
date: 2026-06-01T09:00:00-03:00
draft: false
translationKey: maestro-claude-code-testar-app-no-simulador
category: tecnologia
tags: [maestro, mobile-testing, react-native, claude-code, qa-automation]
author: bruno-raphael
description: "You can have Claude navigate and test your app in the simulator like Playwright does on the web. With Maestro: one YAML, iOS and Android, zero instrumentation."
cover: covers/maestro-claude-code-testar-app-no-simulador.png
cover_alt: "Blog Nextside cover for a post about testing apps in the simulator, with the Nextside N symbol in red on a dark background"
cta:
  servico: discovery
  posicao: ambos
  contexto: "Want to put Claude to test your app but not sure it fits your stack? We validate the setup with you in a Discovery."
canonical: ""
---

## TL;DR

Claude Code already navigates your site on its own through Playwright: it clicks, fills, validates regressions. For mobile apps you can do the same thing, but nobody really explains how. I went digging. The answer is <dfn>Maestro</dfn>, an open source mobile E2E testing framework with flows written in YAML, plugged into Claude Code. A single test file runs the same on iOS and Android, on top of the compiled binary, without instrumenting the app. React Native, native, or Flutter, doesn't matter. Claude inspects the screen, writes the flow, runs it, and fixes what breaks. And no: the right path is NOT "giving Claude access to the screen". Screenshot by coordinate is the last resort, not the first.

This post is the setup I put together to close on mobile the gap that [Playwright already closed for the web](/en/posts/2026/05/16/mcp-playwright-validacao-local-com-qualidade/). [Here at Nextside](https://nextside.tech) it's not a production pipeline yet. It's the path I'm adopting, with the engineering broken down, the commands in hand, and the numbers from people who already walked it.

## It's not "giving access to the screen". It's reading the tree.

When I bring this up, the same question always comes, and I asked it myself at first: "can't Claude just look at the screen and tap, like a human?". It can. It's called <dfn>Computer Use</dfn>: Claude controls the interface through screenshots and clicks on pixel coordinates. It launched in Claude Code in March 2026, it drives the simulator, it works for a demo.

But it's the wrong way for testing.

The Playwright you already use never looked at a single pixel. It reads the *accessibility tree*, the structured tree that describes "button labeled Sign in, here". It acts by element, not by coordinate. That's why it's fast and doesn't hallucinate where to click.

The difference is measurable in tokens: the accessibility tree of one screen comes in at [around 10 tokens, and a screenshot of the same screen costs 1,600 to 6,300](https://github.com/conorluddy/ios-simulator-skill). Multiply that by every step of a twenty-screen test and you get why vision doesn't scale in a QA loop.

At the bottom there are three ways to make Claude touch the app, from best to worst:

- **MCP or CLI reading the tree.** Structured, deterministic, cheap on tokens. It's the "Playwright way", and it's where Maestro lives.
- **Computer Use through screenshots.** Claude sees the screen and guesses a coordinate. It generalizes to any app, but it's slow (2 to 5 seconds per action), misses clicks, and burns context.
- **Nothing.** You testing everything by hand, which is what we're leaving behind.

Anthropic itself orders it this way. The [Claude Code tool hierarchy](https://code.claude.com/docs/en/computer-use) is MCP first, then shell, then Chrome, and it only falls to screen control when nothing else reaches: "native apps, simulators, and tools without an API".

**Screenshot is the last resort, not the first.**

## Maestro: one YAML, iOS and Android, zero instrumentation

If the right way is to read the tree, I need a tool that exposes the simulator's tree to Claude. There are several. I settled on Maestro, and for anyone keeping React Native and native apps, it wins on three concrete counts:

- **It works at the accessibility layer, on top of the compiled binary.** It doesn't matter if the app is React Native, native Swift/Kotlin, or Flutter. Maestro tests the finished APK/IPA, with no driver installed and no change to the source code. For a team running RN and native side by side, that's the end of maintaining two testing stacks.
- **The same file runs on both systems.** You write the flow once. It runs on the iPhone simulator and the Android emulator without rewriting a single line.
- **YAML that humans and machines read.** It's not code with fragile selectors. It's a declarative sequence that Claude generates and edits on the spot.

A Maestro flow starts simple like this:

```yaml
appId: com.yourcompany.app
---
- launchApp
- tapOn: { id: "login_button" }
- inputText: "user@nextside.tech"
- tapOn: "Sign in"
- assertVisible: "Welcome"
```

`appId`, three dashes, and the commands in almost natural language: `launchApp`, `tapOn`, `inputText`, `assertVisible`. Anyone who's never seen it gets it in ten seconds.

Where it gets serious is reuse. The login repeats in every test, so you pull it out once and call it with `runFlow`:

```yaml
# flows/login.yaml
appId: com.yourcompany.app
---
- launchApp: { clearState: true }
- tapOn: { id: "login_button" }
- inputText: "user@nextside.tech"
- tapOn: "Sign in"
```

```yaml
# flows/checkout.yaml
appId: com.yourcompany.app
---
- runFlow: login.yaml          # reuses the whole login
- tapOn: { id: "product_42" }
- scrollUntilVisible:
    element: { text: "Checkout" }
- tapOn: "Checkout"
- assertVisible: "Order confirmed"
```

Change the login rule in one place, it holds across the twenty tests that call it. Notice the `scrollUntilVisible` and the `clearState: true`: Maestro has a command to scroll until it finds, clear state, change permission, set location. And it **waits for the element to show up on its own**, without you scattering `sleep` across the test. Sleep is a smell of a badly written test, here you don't need it.

**Same file. iOS and Android. Without touching the app's code.**

## From zero to the first test

The real "how to use it" starts before Claude. You need three things on the machine:

- **Java 17 or newer.** Maestro's engine runs on the JVM. Check with `java -version`.
- **Xcode and the Command Line Tools.** That's what unlocks the iOS simulator.
- **Android platform-tools with `$ANDROID_HOME` set and an emulator running.** Check with `adb devices`.

With that in place, install Maestro in one command:

```bash
curl -fsSL "https://get.maestro.mobile.dev" | bash
# or, on macOS, via Homebrew:
# brew install mobile-dev-inc/tap/maestro
maestro --help   # confirms it's alive
```

Boot a simulator (or emulator), install your app on it, and run the flow:

```bash
maestro test flows/checkout.yaml     # one flow
maestro test flows/                  # the whole folder
```

That alone already gives you E2E tests running locally, with no AI at all. AI comes in so you stop writing these YAMLs by hand.

## The loop in practice: Claude writes the test looking at the app

Connect Maestro to Claude Code in one command:

```bash
claude mcp add maestro -- maestro mcp
```

That hands Claude a handful of tools: `inspect_screen` (grabs the screen's view hierarchy as compact JSON), `run` (executes a flow), and `open_maestro_viewer` (embeds the simulator in a window where you watch each command run in real time).

The loop this unlocks changes the game:

1. **Claude inspects** the screen live. It reads the tree, it doesn't guess.
2. **Claude writes** the flow YAML, [without you hunting for element IDs by hand](https://maestro.dev/blog/maestro-mcp-an-introduction).
3. **Claude runs** it on the simulator.
4. **Claude diagnoses** what failed by looking at the hierarchy, and fixes the test itself.

Step 4 is the one that saves the most sanity. When a `tapOn: "Sign in"` breaks because the button became "Log in" in a refactor, the manual flow is: test fails in CI, someone opens it, finds out, fixes the selector, pushes again. With the loop, Claude rereads the hierarchy, sees the label changed, switches to the stable `id`, and shows you the diff. You approve it or not. [Maestro calls this self-healing](https://maestro.dev/blog/maestro-mcp-an-introduction). It's test maintenance, the most tedious part of QA, coming off your back.

In React Native, what makes this loop reliable is the `testID`. The one you already put on your components becomes Maestro's `id` directly:

```jsx
<Button title="Sign in" testID="login_button" onPress={onLogin} />
```

Prefer `testID` over text, always. Text changes with translation and with copy revisions. The `testID` only changes if you change it on purpose. And when you don't know which selector exists on a screen, `maestro studio` opens a visual inspector in the browser: you click the element, it shows the available selectors and generates the YAML for the step. That's how you teach Claude to aim at the right places in your app.

### MCP or Skill+CLI: which to use?

Both work. The choice is about context. The **MCP** is plug-and-play: one command and Claude has the tools. The price is that every MCP loads the tools' schema into the model's context, and that eats tokens every session.

The alternative is a **Skill** that teaches Claude to run `maestro test flow.yaml` straight in the terminal. Leaner, because you don't pay the server overhead. The [community itself is migrating from MCP to Skill+CLI for this reason](https://news.ycombinator.com/item?id=45642911). My rule: I start on the MCP to explore and prototype fast. Once the flow becomes routine, I wrap it in a Skill with the CLI and drop the server.

{{< cta servico="discovery" variante="mid" />}}

## The iOS toll (the part nobody posts)

Now the honest part, because selling this as magic is a disservice.

First: AI-generated tests get it right [70 to 80% on the first pass](https://medium.com/coding-nexus/someone-built-a-tool-that-lets-claude-code-autonomously-test-your-entire-ios-app-5256d0a49703). Claude picks the wrong selector, forgets a wait. The flow that works is letting the AI generate v1, running it once to validate, and handing maintenance back to it. It's not "send it and forget it".

Second, and heavy for anyone in mobile: **iOS charges a toll.** A dev documented setting up the same QA on both platforms. [Android took 90 minutes, iOS went past six hours](https://christophermeiklejohn.com/ai/zabriskie/development/android/ios/2026/03/22/teaching-claude-to-qa-a-mobile-app.html). His line sums up the whole decade of mobile automation. "Android hands you a WebSocket and says: here's the app, do whatever you want. iOS hands you a locked door and a note asking you to use Xcode."

The good news is that Maestro abstracts away a good chunk of that toll, it's the same `tapOn` on both. But two stones you'll still step on in React Native:

- **Nested component on iOS.** iOS "swallows" the tap when you have a `Text` inside a `TouchableOpacity` inside another tappable container. The fix is `accessible={false}` on the outer container and `accessible={true}` on the inner element. It's annoying, but it's once per component.
- **Expo Go doesn't accept `launchApp`.** Running through Expo Go, the app lives inside the Expo container, and `launchApp` with your `appId` won't catch. You have to use `openLink` with the dev URL, or do a real development build (EAS). On bare React Native, `launchApp` works normally.

> "You're going to let a bot write and run the app's tests? This is going to go wrong."

It'll go wrong if you treat the generated test as truth and walk away. It won't if you treat it as a draft the senior reviews, just like you already do (or should do) with code the AI writes. Maestro still hands you the versioned YAML: you can read it in the PR, disagree, fix it. The test is still yours. Claude just stopped making you type it from scratch.

## From a loose test to a routine

A test you run by hand when you remember isn't a safety net. It's theater. The real gain shows up when the flow becomes an automatic routine. Since Maestro is just a command-line binary, it goes anywhere that runs a shell:

```bash
maestro test flows/    # runs the whole suite; exits with an error code if it breaks
```

That `maestro test flows/` is the same line you run locally, in GitHub Actions on every PR, or in a nightly cron. That dev from the real case [left the suite running as a scheduled task every morning at 8:47](https://christophermeiklejohn.com/ai/zabriskie/development/android/ios/2026/03/22/teaching-claude-to-qa-a-mobile-app.html): it boots both simulators, sweeps the screens, analyzes, and files a report on whatever looks broken. The dev wakes up with QA already done.

The cycle closes here. Claude writes the flow looking at the app, the flow becomes a versioned file, the file runs in CI. The AI builds the net, the machine pulls it every night.

## The AI writes the code and the test. You still decide what "works" means.

We [already talked here about the AI reviewing code but not testing software](/en/posts/2026/05/25/code-review-novo-gargalo-coderabbit/). Still true, with a new asterisk: now it TESTS, in the simulator, navigating the app like a user would. What it doesn't do is decide what counts as "worked".

That judgment is yours. The acceptance criteria are yours. Maestro and Claude take the tedious part off your hands: booting the simulator, hunting for the button's ID, typing the flow, running on both systems, fixing the selector that changed. They give back the time for the one thing the machine doesn't do: looking at the app and deciding if it's good.

A good tool doesn't replace judgment. It just removes the excuse of not having tested.
