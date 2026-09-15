# Claude Code Function Hooks — a guided starter

A beginner-friendly guide to **function hooks**, a beta Claude Code feature that turns your safety rules from polite requests into enforced code. This repo is built to be **handed to Claude Code itself**: open it in Claude Code, say *"walk me through this"*, and the agent will explain each hook idea and ask which ones you want before building anything.

## Credit — where this comes from

Everything here is based on **["Claude Code Function Hooks"](https://edge.blueprintgtm.com/p/claude-code-function-hooks)** by **[Ray Amjad](https://www.rayamjad.com/)** ([YouTube](https://www.youtube.com/@ramjad)), published as a guest post on **[On the Edge by Blueprint](https://edge.blueprintgtm.com/)**, Jordan Crawford's newsletter. The original includes a 13-minute video of Ray building hooks live. This repo paraphrases the ideas as a hands-on catalog; read and watch the original for the full walkthrough — it's worth it.

## The problem, in one paragraph

You write rules for Claude in `CLAUDE.md`: "never delete from the database", "never commit secrets". Claude follows them — *most* of the time. But rules in a prompt are text competing with everything else in the context window. Forty turns into a session, one lazily-worded prompt can override them. A rule written as a **hook** is code that runs at a fixed point in the agent's loop. It applies on turn one and on turn eighty. It doesn't get tired.

## What function hooks add

Claude Code has had shell hooks for a while — small scripts that can allow or block an action. Function hooks (beta) go much further. When Claude is about to do something, a function hook can:

- **Block** it, with a reason Claude can read and adapt to
- **Fix** it — rewrite the action into a safer version and let that through
- **Answer** it — return a result itself instead of running the tool
- **Ask you** a question with real buttons, and wait
- **Draw** UI: a pinned status line, a card, a toast
- **Remember** things across turns and across restarts
- **Judge** with a small model when a regex can't decide
- **Extend** Claude with new tools that have the rules built in

You don't code them by hand. You describe the rule in plain English and Claude writes the hook.

## How to use this repo (the intended flow)

1. Clone it and open Claude Code inside it:
   ```bash
   git clone https://github.com/danieluszta/claude-code-function-hooks-guide.git
   cd claude-code-function-hooks-guide
   claude
   ```
2. Say: **"walk me through this repo"**.
3. The agent (following [CLAUDE.md](CLAUDE.md)) will explain what function hooks are, then go through the [hooks catalog](hooks-catalog.md) **one group at a time**, explaining what each hook does and what could go wrong without it.
4. You pick the ones you want. The agent builds them one at a time and shows you how to test each.

The agent is explicitly instructed **not** to install everything silently. If it starts building without asking, tell it to read `CLAUDE.md` again.

## Quick reference: enabling the feature

Function hooks are in beta as of September 2026, behind an environment variable:

```bash
CLAUDE_CODE_ENABLE_FUNCTION_HOOKS=1 claude
```

Then, inside Claude Code:
- `/plugin-authoring` — the skill that writes function hooks from a plain-English description
- `/reload-plugins` — required after every change; without it nothing happens

Hooks can live at **project level** (this repo only) or **user level** (every project on your machine). Decide per hook — a database write-gate probably belongs at user level; a project-specific status line does not.

## Honest limits (from the original post)

- **It's beta.** Shapes may change between releases. If a hook breaks after an update, ask Claude to rebuild it.
- **Function hooks fail open.** If one crashes, Claude Code skips it, and repeated crashes unload the plugin. Keep your hard "never run this" rules as classic shell hooks too — those fail closed.
- **Hooks have a time budget.** Slow external calls can get cut off (waiting for *your* answer doesn't count).
- **Hooks see tool calls, not script internals.** A 40-minute script is invisible until it ends. Chunk long jobs into batches, or have the script write a progress file.
- **Model-judged hooks add latency.** Put them on slow events (turn end, plan submission), not on everything.

## License

MIT for the contents of this repo. The original article and video remain © their authors — go subscribe to [On the Edge by Blueprint](https://edge.blueprintgtm.com/) and check out [Ray Amjad's Claude Code material](https://www.rayamjad.com/).
