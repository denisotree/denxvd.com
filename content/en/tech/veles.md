---
title: Veles — My Multi-Agent Framework With Memory That Gets Smarter Every Session
date: 2026-06-19T14:00:00
Description: "I built Veles — a minimal, local-first multi-agent framework that keeps structured memory for every project, works with any LLM provider, and accumulates skills and tools as you go. I explain why I wanted my own framework instead of off-the-shelf chat tools, what Veles can do, and how to give it a quick try."
tags:
  - veles
  - llm
  - agent
  - python
  - cli
  - linux
  - macos
  - review
Categories:
featured_image: "/tech/veles/veles.jpg"
featured_image_caption: ""
summary: "I built Veles — a minimal, local-first multi-agent framework that keeps structured memory for every project, works with any LLM provider, and accumulates skills and tools as you go. I explain why I wanted my own framework instead of off-the-shelf chat tools, what Veles can do, and how to give it a quick try."
DisableComments: true
draft: false
---

![Veles TUI showing an answer grounded in the project's memory](/tech/veles/veles.jpg)

[Veles](https://github.com/denisotree/veles) — a minimal multi-agent framework that keeps structured memory for every project and gets more useful with every session. It runs right in your terminal, works with any LLM provider — cloud or local — and accumulates knowledge, skills, and tools as you go. I built it first and foremost for myself: I work with LLMs a lot, and a recurring problem is that the whole conversation doesn’t fit in the context window — and after compaction the model forgets important insights, so you keep reminding it of the rules, events, and documents and rebuilding context. I wanted a framework that helps agents remember my decisions, stores memory well, and — if I want — runs entirely on a local model, without being tied to a single vendor.

I [work in the terminal]({{< relref "data-analyst-cli" >}}) a lot and like my tools to be convenient. Veles grew out of a need to bring together, in one place, the features scattered across different utilities and frameworks for working with agentic systems. In this article I’ll explain why I built it, what it can do, and how to give it a quick try.

---

### Why Not an Off-the-Shelf Chat Agent

True, there’s Claude Code, OpenCode, OpenClaw, Hermes (the closest to what I wanted — though the way its code is organized horrifies me), and a dozen cloud agents — and they really are great. But I wanted the best bits of each in one box — the ideal balance, to my mind:

1. **Memory, not amnesia.** So the agent remembers the project context between sessions and I don’t explain the same thing over and over.
2. **Local-first.** So memory and state live on my machine — no telemetry, no cloud sync — and, if I want, inference too, on a local model. This matters when you work with sensitive data.
3. **No vendor lock-in.** So the same agent can talk to any model — Claude over the API, or a local one via Ollama.
4. **A clean core.** Minimal code with no god-files, the kind you’re not afraid to extend yourself.

Some off-the-shelf tools cover one piece but not all of it at once. Others — like OpenClaw or Hermes — go the opposite way and bundle everything out of the box, even what you don’t need. I built Veles on the inverse principle: a minimal core, with everything else easy to plug into it as modules.

---

### What Veles Can Do

1. **Memory that compounds**  
   This is the headline feature. After every session the Curator distills it into insights, rules, and digests that live in `.veles/` right inside the project. From there the agent recalls relevant facts and past decisions on its own — you stop cycling through the same context. Insight extraction catches phrases like “always do X” or “never do Y” and saves them as persistent project rules.

   ![Veles: asking a question in the TUI and getting an answer from the project's memory](/tech/veles/tui-hero.gif)

2. **Any provider, no vendor lock-in**  
   OpenRouter, Anthropic, OpenAI, Gemini, Ollama, llamacpp, or your `claude` / `gemini` CLI subscription — no API key needed. Different task types can route to different models: planning to a stronger model, compression and insight extraction to a cheaper one.

   ```bash
   # Route task types to models: set it once and forget it
   veles route set compressor anthropic/claude-haiku-4-5
   veles route show
   ```

3. **Local-first and sandboxed**  
   No telemetry, no cloud sync — the agent sees only the active project directory, and attempts to escape it (symlinks, `..`) are blocked. Every sensitive call (running a shell, writing files, fetching a URL) triggers a “trust ladder” prompt. And if you want nothing to leave your machine at all, run a local model via Ollama — then the whole loop, inference included, stays with you.

4. **A knowledge base right in the project**  
   By default, `veles init` scaffolds an LLM wiki, but you can pick another layout. From there the agent maintains the project according to the chosen layout — or doesn’t touch the structure at all, say for a code repo. In wiki mode you can hand it a source, and it turns it into a page it later cites in its answers:

   ```bash
   # Add a source (PDF, web page, text) → it becomes a wiki page
   veles add paper.pdf
   veles run "What do we know about the authentication design?"
   ```

   ![Veles ingesting a source into the LLM wiki, then citing it in an answer](/tech/veles/kb-ingest.gif)

5. **Skills and tools that accumulate**  
   A skill is a reusable prompt-block (`SKILL.md`) that automatically becomes an agent tool. A good skill can be “promoted” from a project to user-global — and then it’s available everywhere. Built-in dedup finds near-duplicate skills before they multiply.

6. **A TUI with modes**  
   `veles tui` opens an interactive REPL. Slash commands surface everything live — `/status`, `/tokens`, `/context` — and `Shift+Tab` cycles modes (auto / planning / writing / goal).

   ![Veles TUI tour: slash commands and switching modes](/tech/veles/tui-tour.gif)

7. **Daemon and channels**  
   Veles can run as a persistent daemon with an HTTP/WS API that external channels connect to — messengers like Telegram or Slack, or your own integrations. Telegram works out of the box: put a bot on top and chat with your project’s agent right from your phone, with a persistent session per user. Other channels are easy to build on top of the same API.

   ![Veles TUI daemon and channels control panel](/tech/veles/tui-daemon.gif)

8. **Long-running tasks**  
   When a task takes more than one pass, there are goals with a budget, scheduled jobs, and multi-step research.

   ```bash
   # A long goal with a money limit
   veles goal start "Migrate the auth module to the new provider" --max-cost-usd 2.00

   # Deep research: the question is split into parallel sub-questions
   veles research "What are the trade-offs between SQLite and PostgreSQL for this use case?"
   ```

---

### How to Try It

Veles installs via [uv](https://docs.astral.sh/uv/) (Python 3.13+ required):

```bash
# The package is published as veles-ai; the command is veles
uv tool install veles-ai
```

```bash
# An API key — I recommend OpenRouter: all models with one key
export OPENROUTER_API_KEY=sk-or-v1-...
```

```bash
# Create a project and ask the first question
veles init && veles run "Read the README and describe the project"

# Or jump straight into the interactive mode
veles tui
```

If you’d rather build from source — `git clone`, then `uv tool install .` from the repository directory.

---

### Where Veles Isn’t the Right Fit

I’ll be honest: this isn’t a replacement for big agentic IDEs like Claude Code or Cursor. Veles is a minimal core built around my workflow — memory, the learning loop, and the provider protocol — while almost everything else (TUI, daemon, Telegram, research, scheduler) is an optional module you turn on deliberately.

A couple more honest caveats. The project is young — version `0.6.x`, Beta as of this article — and it requires Python 3.13+, which not everyone has yet. And like any LLM agent, Veles sometimes gets it wrong: memory and the trust ladder help a lot, but it’s a force-multiplier, not an autopilot you can blindly trust. Answer quality depends directly on the model you pick.

---

### Plans and Wrap-up

Veles has already become the thing I run more and more of my daily tasks through — and the longer I use it, the more noticeable the effect of its accumulated memory. The project is open source (Apache-2.0) and built with reliability in mind: over 3200 tests, all of them running without the network. In future articles I’ll dig into specific pieces in more detail: how project memory and the learning loop work, routing task types to models, and how to write your own skills and tools.

### Feedback

If you have ideas for improvements or you found a bug — open an [issue on GitHub](https://github.com/denisotree/veles/issues) or message me (LinkedIn / Instagram / Telegram).
