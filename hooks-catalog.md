# The hooks catalog

Sixteen function-hook ideas, paraphrased from [Ray Amjad's post on On the Edge by Blueprint](https://edge.blueprintgtm.com/p/claude-code-function-hooks). They fall into four jobs: **block the dangerous action, count the spend, show the progress, remember then ask.**

Each entry has: what it does, why a beginner should care, and a plain-English description you (or your agent) can paste into `/plugin-authoring`.

Risk key: 🔴 protects you from data loss · 🟡 protects your wallet · 🟢 quality of life

---

## Job 1: Block the dangerous action

### 1. Write gate 🔴
**What:** All database/CRM writes (insert, update, delete, merge) are denied by default. The first attempt in a session asks you whether to enable writes; your answer holds until you close Claude Code.
**Why:** This is the single biggest protection. One misread prompt can no longer touch your data without you pressing a button first.
**Builder prompt:** "Deny any write, update, delete or merge through my database and CRM tools by default. On the first attempt, ask me with buttons whether to allow writes for this session. Remember the answer until the session ends."

### 2. Count before you write 🔴
**What:** Before any SQL `UPDATE` or `DELETE` runs, the hook runs the matching `SELECT COUNT` first, shows you the number of affected rows, and asks. Statements with no `WHERE` clause are refused outright.
**Why:** "Delete the test rows" that matches 40,000 rows gets caught before it runs, not after.
**Builder prompt:** "Before any SQL UPDATE or DELETE, run the matching SELECT COUNT and show me the number. Ask before running the real statement. Refuse any UPDATE or DELETE that has no WHERE clause."

### 3. Backup before bulk 🔴
**What:** Bulk deletes or merges are refused until the affected records have been exported to a CSV in a backups folder. The confirmation shows you the backup path.
**Why:** Turns an irreversible operation into a reversible one.
**Builder prompt:** "Before any bulk delete or merge in my CRM, export the affected records to a CSV in a backups/ folder. Refuse the operation until the export exists, and show me the file path when asking for confirmation."

### 4. Read-only subagents 🔴
**What:** Any write attempted by a subagent is denied with the reason "subagents are read-only". The main session can still write (through hook 1's gate).
**Why:** Subagents run with less of your context and less of your attention. They should gather, not change.
**Builder prompt:** "Deny any tool call from a subagent that writes to the database or CRM, with the reason 'subagents are read only'. The main session may still write."

---

## Job 2: Count the spend

### 5. Sample before the batch 🟡
**What:** Any paid API call that runs over a list requires a 10-row sample to have run first in the session. Then it shows estimated total cost and gives three buttons: run 10 more, run all, stop.
**Why:** You find out the enrichment is broken (or expensive) after 10 rows, not after 60,000.
**Builder prompt:** "Before any enrichment or lookup call over a list, require that a 10-row sample has run in this session. Then show rows × cost-per-row and give me three buttons: run 10 more, run all, stop."

### 6. Spend ledger 🟡
**What:** Keeps a running tally of every paid API call, multiplied by unit prices you configure. A pinned line above the prompt shows today's spend per vendor.
**Why:** Agent workflows spend money invisibly. This makes the number ambient.
**Builder prompt:** "Keep a ledger of every call to my paid APIs. Multiply by unit prices I set in the plugin config. Pin one line above the prompt showing today's spend per vendor."

### 7. The threshold ladder 🟡
**What:** Before a batch runs, estimate cost from row count × unit price. Under $100: print it and continue. $100–$1,000: ask. Over $1,000: make you type the amount to confirm.
**Why:** Friction proportional to the stakes. Small runs stay fast; big runs can't happen half-asleep.
**Builder prompt:** "Before any batch runs, estimate the cost from row count and unit price. Under 100 dollars, print it and continue. Between 100 and 1,000, ask me. Over 1,000, require me to type the amount to confirm."

### 8. Daily subagent cap 🟡
**What:** Tracks subagents spawned today across all sessions. At 80% of your cap, new subagents are forced onto a cheaper model. At 100%, new spawns are denied with a reason Claude can read.
**Why:** Prevents a runaway multi-agent loop from becoming a runaway bill.
**Builder prompt:** "Track subagents spawned today across sessions. At 80 percent of my daily cap, force new subagents onto the cheapest model. At 100 percent, deny new spawns with a readable reason."

*Note: hooks can't see your token bill directly — no event carries a token count. They see each paid call as it happens and multiply by prices you set.*

---

## Job 3: Show the progress

### 9. Kickoff card 🟢
**What:** Starting any pipeline draws a card: source file, row count, vendors involved, estimated cost — with buttons for sample 10 / run all / stop. Nothing starts until you press one.
**Builder prompt:** "When I start a pipeline run, draw a card with the source file, row count, vendors and estimated cost, with three buttons: sample 10, run all, stop. Do not start until I press one."

### 10. Pinned progress line 🟢
**What:** Registers a batch-processing tool (e.g. 100 rows per call) and pins a live line under the prompt: rows done / total, spend so far, failures, last error.
**Why:** Also the workaround for "hooks can't see inside a long script" — chunking work into tool calls makes it countable.
**Builder prompt:** "Register a tool called enrich_batch that processes 100 rows per call. Count calls and pin a line under the prompt: rows done over total, spend so far, failures, last error message."

### 11. Catch-up card 🟢
**What:** If more than 5 minutes passed since your last prompt, your return draws a card: minutes away, turns completed, files changed, failed tool calls. Then it clears.
**Builder prompt:** "If more than 5 minutes passed since my last prompt, draw a card when I return: minutes away, turns completed, files changed, tool calls that failed. Then clear it."

### 12. The receipt 🟢
**What:** Every turn that ran a pipeline ends with a receipt under the answer: rows processed, spend, failures, output file path.
**Builder prompt:** "At the end of every turn that ran a pipeline, append a receipt: rows processed, spend, failures, and the output file path."

---

## Job 4: Remember, then ask

### 13. Park the risky moves 🔴
**What:** If you've been idle for 10 minutes, actions that send email, delete rows, deploy, or spend money get parked instead of run. Safe work continues. When you return: a list of parked actions with run / drop / later. The list survives restarts.
**Why:** The agent keeps working while you're at lunch — but nothing irreversible happens without you.
**Builder prompt:** "If I have not typed for 10 minutes, park any tool call that sends email, deletes data, deploys or spends money. Let safe work continue. When I return, list the parked actions and ask: run, drop, or later. Persist the list across restarts."

### 14. Read it before thousands do 🔴
**What:** Any command that sends a bulk email or newsletter requires a preview file first. The hook estimates reading time and refuses to send until that much time has passed since the preview was written *and* you type a code hidden on its last line.
**Why:** Proof that a human actually read what's about to hit thousands of inboxes.
**Builder prompt:** "Before any command that sends a sequence or newsletter, require a preview file. Estimate reading time at one second per four words. Refuse to send until that time has passed since the preview was written and I type the 4-letter code you placed on its last line."

### 15. Arithmetic checker on every plan 🟡
**What:** Before a plan is shown for approval, a small model checks that unit price × count equals the headline total. Mismatches bounce the plan back with both numbers side by side.
**Builder prompt:** "Before showing me a plan for approval, have a small model verify that price per item times item count equals the headline total. If they disagree, bounce the plan with both numbers shown."

### 16. Audit log 🟢
**What:** After every turn, one log line per tool call: time, tool, input, allowed or denied — written to a project file (optionally also shipped to a log endpoint).
**Why:** For anyone in a regulated industry, or anyone who wants to review what the agent actually did.
**Builder prompt:** "After every turn, write one log line per tool call — time, tool, input, allowed or denied — to a log file in my project."

---

## The meta-move: let Claude find YOUR hooks

The best hooks are the ones your own history is asking for. Every correction you've typed twice is a rule that should be code. Ray's suggested approach — don't ask "what hooks should I make?", ask Claude to mine your actual transcripts:

> `/plugin-authoring` Look through my previous Claude Code transcripts. Find the rules I keep repeating, the commands I keep blocking, and the questions I keep answering. Give me 20–30 function hooks I could create for my most-used workflows, ranked by how often the problem appeared.

Pick the top five it finds and build those first.
