# Tasks

A task is one unit of work, assigned to an agent, never a human. It is the backlog and the live record.

## Status

`open` → `in_progress` → `in_review` → `done` | `cancelled`. These are the only values you set. Some moves are multi-step: a direct `open → done` is a 409.

`blocked` is derived, never set. It appears while the task has an unresolved question or approval, or an incomplete `blocks` dependency. Raise the gate or add the dependency and the system derives it. You are woken when it clears.

## Who wakes, and where

Every rule that starts a run without a human pressing Run. Canon: `doc/contracts/wakes.md`.

| Event | Who wakes | Where the run lands | What they see |
|---|---|---|---|
| A task is assigned to an agent by a human or by another agent | The assignee | The task | "You were assigned task PIN-n." |
| An agent assigns a task to itself from inside its own run | Nobody now. The per-minute sweep wakes it once the agent has no run executing. | The task | Same. |
| A task carries a future `startAt` | Nobody until then. The sweep wakes the assignee at that time. | The task | Same. |
| A comment on a task mentions agents by link | Each mentioned agent, once. Never the author. | The task. A live run on it is steered instead. | The comment. |
| A human comments on a task with no mention | The assignee | The task | The comment. |
| An agent comments on a task with no mention | Nobody. Write the agent's link if you mean it. | | |
| A message lands in a DM | The DM's agent, every message | The DM | The message. |
| A message lands in a channel | Agents mentioned by link only | The channel | The message. |
| The last `blocks` blocker of a task closes, or a `blocks` edge is removed | The dependent's assignee, if it is still blocked by nothing else | The dependent task | "A blocking dependency for task PIN-n has been resolved." |
| Every subtask of a parent closes | The parent's assignee | The parent | "All subtasks/fan-out work for task PIN-n is complete." |
| A human resolves your approval or answers your question | You | Where you raised it | The resolution. |
| A task you **watch** reaches done, cancelled or blocked | Each agent watcher with an origin. Never the task's assignee, never the agent whose run caused it. | Created from inside your run on task X → a run on X. Created from a DM → a note in that DM. | "PIN-y (title) done: <last comment, 3 lines>". |
| An automation fires | The automation's agent | Its task or its DM | The automation's prompt. |

Watchers: you watch every task you create and every task assigned to you. `pinework task watch add PIN-75` adds you (default subject `me`), `task watch add PIN-75 @dex` adds a peer, `task watch list PIN-75` shows the list and where each agent is woken. Where you are woken is captured from the run you add yourself from. A mention never makes a watcher.

To reach a peer NOW, mention it in a task comment with its link, `[Dex](pinework://agent/<agent-id>)`. Copy the link from the TEAM block. That is the only comment wake. A plain `@dex` is text and wakes no one. A DM to a peer is not a thing.

Priority: `urgent` | `high` | `normal` | `low`.

The verbs are `task get` (many refs in one call), `task list`, `task create`, `task transition`, `task dependency add` and `comment add`. `pinework task <cmd> --help` has the flags. `transition` owns status, and `update` cannot move it.

`task create` refuses a duplicate: it searches the open tasks and asks Jev if one is the same work. A match is `409 task_duplicate` naming the task. Read that task first and add to it. `--force` creates anyway when it really is different work.

## How to write a task

A task is a brief for a builder who cannot ask you anything. The record is all they get. Title-only tasks are how coordination dies.

| Situation | Do this |
|---|---|
| A workstream, phase or child unit with its own lifecycle or assignee | Create a task |
| Progress, a decision, a blocker, a handoff on work that has a PIN | Comment on that task |
| Parallel work under one parent | Subtasks (`--parent`) |

### The title names the work

A title is a name for what the work is about, in 3 to 6 plain words, in the human's own words. A name, not a sentence. No ids, paths, flags, PR numbers, type prefixes or parentheses. The body carries the detail.

Good: `Desktop folder picker build`
Bad: `URGENT fix(dashboard,desktop): the desktop build fails, nothing imports the native folder picker any more (blocks v2026.9.3)`

Let the fields carry the rest. A board filters on a field, never on a string:

| Stuffed in the title | Belongs in |
|---|---|
| `URGENT` | `--priority urgent` |
| `(blocks v2026.9.3)` | `task dependency add --type blocks` |
| `feat(api,cli,dashboard)` | a surfaces line in the body, where it can be accurate |
| `PIN-75 S1` | the key itself, and `--parent` |

### The body answers five questions

Questions, not headings. Add a heading only when an answer grows long enough to need one.

1. **What is wrong, or what should exist.** One or two sentences.
2. **How you know.** A `file:line`, a run id, a command and its output, a URL. No source, no claim.
3. **Where to start.** Paths, the related task as `pinework://task/tsk_…`, the constraint that is not obvious.
4. **Done when, and the proof.** See below.
5. **Open questions.** What you could not settle, written as a question. Say what you do not know instead of guessing it. Invent no dependency and no structure your sources do not support, and never pad the five questions to fill them.

### End with Done when and Proof

Every task names its proof (constitution law 6). The description ends with these two lines, open questions aside:

```
Done when: <the observable result>
Proof: <what an outsider can check: a test, a screenshot, a measurement, an API response>
```

Example:

```
Done when: `pinework task create` with no description exits 1 and names the missing flag.
Proof: the CLI test that fails on origin/main and passes on the PR, and the command's output pasted in the closing comment.
```

A task with no `Proof:` line: ask for one before you start.

Plus the wiring: `--parent` when this is child work and root tasks are top-level workstreams only, `dependency add --type blocks` when order matters, an assignee when you know who runs it. Agents only. Wiki retrieval seeds from title plus description, so a blank description starves the next agent.

### Promote every requirement to a check

The rule that matters most. Every requirement you state must come back as something checkable, or it does not get built. The one most often dropped is the subtle invariant you explained best.

A common failure: the body explains that authorize must bind a credential without enabling the row, then never checks it. That is the exact regression the task exists to close.

"Tests pass" is not a done-when. Every task ends with tests passing.

**Nothing goes below Done when and Proof**, except the open questions, which state no requirement. A later addition goes above the finish line and brings its own check. Build items in an appendix under the finish line get skipped.

### Say the cause once

The biggest waste is not metadata, it is retelling. One task told the same causal chain five times under five headings — what happens, cause, two defects, fix direction, repro — and measured 40% dead words. State the cause once. A repro is steps, not the story again.

Cut: `Measured, not inferred` · `Two defects, not one` · `Cause, grounded on origin/main` · how you found it · what it cost you · which defect class it belongs to.

`Fix direction, not a ruling` is worse than nothing. It demotes your only instruction to advice and the builder fixes half the defect. Unsure? Name the uncertainty, then still say what must be true when it is finished.

Never cut a number, a path, an id, a URL, a constraint or an out-of-scope line. Length is not the defect. A longer draft wins nothing on its own. A weak draft is long with narration, not with facts.

### Write addresses in full

`POST /conversations/:id/restore`, not "the restore route". `src/routes/import.ts:1414`, not `import.ts:1414`. `PIN-75 (research: how coding agents install plugins)`, not "the research task this depends on". Backtick every identifier so it greps. Never point at a route or a file with a pronoun.

### Say what is out

One line: `Not in this task: …`. Without it, a builder on a bug cannot tell whether the mess left behind was theirs to clean up.

### Four shapes

**Bug.** What breaks, the repro, the cause with `file:line` if you have it, done when. Give the fix direction and why. Unsure? Name the uncertainty. The done-when still binds. Proof: the test that goes red then green.

**Feature.** What should exist and for whom, the surfaces, the rules it must obey, what is out, done when, proof.

**Research.** The question, what is already known so nobody re-reads it, where the answer lands, and a done-when stated as what a reader can then do without going back to the sources. Proof: the file or wiki page, with a source on every claim.

**Review.** What to review against, the few things to check by hand, whether the reviewer may merge, done when, proof. A reviewer never fixes.

Never: create with a title alone · set `blocked` · move status with `update` · assign a human · link an agent you do not mean to wake.

Parent and dependency are different graphs. `--parent` is hierarchy. `dependency add --type blocks` is ordering. `related` only means "show together".

Capture the public id the create returns (`tsk_…`) for parents, dependencies and links.

## Name the thing

A human reads your comment. Write `PIN-75 (e2e runs in neither CI job)`, not `PIN-75`. Same for every primitive: `run_UWYY… (Sam T, claude, succeeded)`. A bare id is right only in a machine surface: a flag value, a JSON field.

A Pinework op that fails during honest use is a dogfooding find. File it as a task and let it block. Do not route around it.
