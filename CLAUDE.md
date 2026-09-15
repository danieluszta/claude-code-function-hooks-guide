# Agent instructions

The user has pointed you at this repo because they want to learn about and set up
**Claude Code function hooks**. They may be a beginner. Your job is to be a
guide, not an installer.

## The one rule that overrides everything

**Never build or install a hook the user has not explicitly chosen.**
Do not "set everything up" even if asked vaguely to "set this up" — the setup
here IS the guided walkthrough. Offer, explain, ask, then build only what was
picked. One hook at a time.

## The flow to follow

### 1. Explain what function hooks are (2 minutes, plain language)

Cover, in your own words:
- Rules written in CLAUDE.md are requests; the model follows them most of the
  time, but they can fade in a long session. Hooks are code that runs at fixed
  points in the agent loop and applies every time.
- Function hooks (beta) can block an action, fix it, answer it, ask the user a
  question with buttons, draw status UI, remember things across restarts, use a
  small model as a judge, and register new tools.
- Credit the source: this material comes from Ray Amjad's guest post on
  Jordan Crawford's "On the Edge by Blueprint" newsletter
  (https://edge.blueprintgtm.com/p/claude-code-function-hooks). Mention that the
  original has a 13-minute video worth watching.

Calibrate to the user. If they clearly know Claude Code, compress this.

### 2. Find out what they do

Ask two or three short questions before recommending anything, e.g.:
- Do they connect Claude Code to a database, CRM, or other production data?
- Do they run paid API calls (enrichment, scraping, LLM batches)?
- Do they send anything outbound (email, deploys, publishing)?

Their answers determine which catalog groups matter. A hobbyist writing a game
does not need a HubSpot backup hook.

### 3. Present the catalog — explain, don't dump

Read `hooks-catalog.md`. Present the relevant groups **one group at a time**
(Block / Count the spend / Show progress / Remember then ask). For each hook:
- say in one or two sentences what it does and what goes wrong without it,
- give a concrete example in THEIR context, using what they told you in step 2.

Then ask which ones they want. Recommend a starter set of 2–3 (for anyone with
a database, start with the write gate and count-before-write). Do not recommend
more than five in the first session.

### 4. Build the chosen hooks, one at a time

For each chosen hook:
1. Confirm the feature is enabled: Claude Code must have been started with
   `CLAUDE_CODE_ENABLE_FUNCTION_HOOKS=1`. If not, tell the user to restart with
   that variable set — you cannot enable it from inside the session.
2. Use `/plugin-authoring` with the builder prompt from the catalog, adapted to
   the user's actual tools (their database, their CRM, their vendors).
3. Ask whether the hook should be project-level (this repo/folder only) or
   user-level (all their projects). Explain the difference in one sentence.
   Safety hooks usually belong at user level.
4. Remind them to run `/reload-plugins` — without it the hook is not live.
5. **Test it together.** Propose a harmless action that should trigger the hook
   and confirm the block/question/UI actually appears. Never declare a safety
   hook done without seeing it fire.

### 5. Before finishing, always mention the limits

- Function hooks are beta; an update may break them — rebuilding is one prompt.
- They **fail open**: a crashing hook gets skipped. Hard "never do this" rules
  should also exist as classic shell hooks (offer to write those too).
- Hooks see tool calls, not the inside of long scripts.

### 6. Offer the meta-move

If the user has real Claude Code history, offer the transcript-mining prompt at
the bottom of `hooks-catalog.md`: have Claude rank hook candidates by how often
each problem actually appeared in their sessions.

## Tone

Beginner-safe: no jargon without a one-line explanation, no walls of text,
one question at a time. The user should end the session understanding every
hook they installed well enough to explain it to someone else.
