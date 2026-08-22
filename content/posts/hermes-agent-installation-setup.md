---
title: "Installing and Setting Up Hermes Agent on Linux and macOS"
description: "A hands-on walkthrough of installing Hermes Agent, configuring your first model, and troubleshooting common setup issues"
publishDate: "22 Aug 2026"
tags: ["hermes", "ai-agents", "cli", "tutorial"]
---

I've spent the last few weeks playing with [Hermes Agent](https://github.com/NousResearch/hermes-agent), the open-source (MIT licensed) AI agent from [Nous Research](https://nousresearch.com). It's an always-on autonomous agent with cross-session memory, self-generated skills, and support for 200+ models across providers like OpenRouter, Anthropic, OpenAI, Ollama, and Nous Portal. What sold me on trying it was how little friction there is to get started — a single command and you're up and running.

This post documents my install experience on Linux, including a few gotchas I ran into so you don't have to. The same steps apply on macOS.

## Prerequisites

The only real prerequisite is **Git**:

```bash
git --version
```

On Linux you'll also want `curl` and `xz-utils` available (the installer downloads Node.js as a `.tar.xz` archive). On Debian/Ubuntu:

```bash
sudo apt install curl xz-utils
```

You do **not** need to install Python, Node.js, ripgrep, or ffmpeg yourself. The installer detects what's missing and handles it all for you:

- **Python 3.11** via [uv](https://docs.astral.sh/uv/) (no sudo needed)
- **Node.js v22** (for browser automation)
- **ripgrep** (fast file search)
- **ffmpeg** (audio conversion for TTS)

:::note
The desktop app additionally requires `g++`/`build-essential` to compile native modules. I went CLI-only, which skips this entirely.
:::

## Installation

One command does it all:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

The whole thing took about two minutes on my machine. Here's what the installer actually does under the hood:

1. Installs missing dependencies (uv, Python 3.11, Node.js, ripgrep, ffmpeg)
2. Clones the repo into `~/.hermes/hermes-agent/`
3. Sets up a virtual environment
4. Symlinks the `hermes` binary into `~/.local/bin/hermes`
5. Kicks off LLM provider configuration

Your data directory ends up at `~/.hermes/`, which holds your config, skills, sessions, and memory files.

## First launch and configuration

Reload your shell so the new `hermes` command lands on your PATH:

```bash
source ~/.bashrc   # or: source ~/.zshrc
hermes
```

That drops you straight into a chat. Before going further, run the setup wizard:

```bash
hermes setup
```

This walks you through everything at once — provider, model, and tools. If you'd rather configure things individually:

```bash
hermes model          # Choose your LLM provider and model
hermes tools          # Configure which tools are enabled
hermes config set     # Set individual config values
hermes config get     # Inspect individual config values
```

### Picking a model

`hermes model` opens an interactive picker covering OpenRouter, Anthropic, OpenAI, Nous Portal, Ollama, and more. API keys get written to `~/.hermes/.env`.

The fastest path if you don't want to juggle per-provider keys:

```bash
hermes setup --portal
```

A single Nous Portal subscription covers 300+ models plus their Tool Gateway (web search, image generation, TTS, cloud browser) in one shot. I used OpenRouter with a pay-as-you-go key instead since I already had one lying around — both routes work fine.

## Verifying it works

With everything configured, start a chat and give it something simple that exercises the toolchain, like asking it to count the files in a directory or summarize a text file. If tools are wired up correctly you'll see tool calls being invoked rather than just a plain completion.

For a deeper health check:

```bash
hermes doctor
```

It reports exactly what's missing (or misconfigured) and how to fix it. Worth running right after install just for peace of mind.

## Troubleshooting

| Problem | Fix |
| --- | --- |
| `hermes: command not found` | Reload your shell (`source ~/.bashrc`) or check that `~/.local/bin` is on your PATH |
| `API key not set` | Run `hermes model` to configure your provider, or `hermes config set OPENROUTER_API_KEY your_key` |
| Missing config after update | Run `hermes config check` then `hermes config migrate` |

The PATH issue bit me once after switching shells — the installer writes to `~/.local/bin/hermes`, and not every minimal shell profile includes `~/.local/bin` by default. Adding this fixed it:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

When in doubt, `hermes doctor` will tell you exactly what's wrong.

## Updating

Hermes ships fast, so update regularly:

```bash
hermes update
```

It auto-detects your install method (git, Docker, or NixOS) and runs the matching upgrade path.

## Closing thoughts

I was genuinely surprised by how hands-off the install was. Coming from other agent frameworks where you're wrangling virtual environments and env files manually, having the installer handle Python, Node, and provider configuration end-to-end felt almost suspicious. My only real stumble was the PATH issue above, and that was self-inflicted.

If you want to go beyond the basics — messaging gateways for Telegram/Discord, running Hermes as a 24/7 daemon, or building custom skills — the [official docs](https://hermes-agent.nousresearch.com/docs/) cover all of that well. The [GitHub repository](https://github.com/NousResearch/hermes-agent) is also very active if you hit anything unusual.
