# Runs

A run is one execution: agent × task × model × access mode × environment. The run you woke on already holds the task's single active run. There is no checkout.

`pinework run get current` reads your run, `run events` its activity, and `run read` another run's final answer. `current` is your own active run. Never hand-roll a polling loop: `watch` streams run status, task transitions and stalls. You never create a run: message the agent instead (`references/messaging.md`).

When your prompt asks for typed output, write the JSON to a file and record it: `pinework run output --schema review_verdict --file verdict.json`. The API checks it against the named schema and answers 422 with the failing field. Fix the file and call again: the last call wins. Only your own run can record its output.

## The CLI in a cloud run

**A cloud run has a pinned `pinework` on PATH.** It is signed in as you. Use it where a path on disk matters: `pinework file upload ./mockup.png --task PIN-123` attaches an image, which the MCP file tool cannot take. The pin can trail the device CLI, so a verb newer than `pinework --version` answers `Unknown … command`.

The same verbs also reach a cloud run as Pinework MCP tools. Every `pinework …` line in these files names the verb, not the transport.

One verb stays device only: `browser` drives the tab inside Pinework Desktop, which a sandbox does not have.

## On every wake

Your system prompt ends with the task brief (`--- TASK CONTEXT ---`). Your message ends with a `--- WHY YOU WOKE ---` block: the reason, the triggering comment or resolved gate, then the current message. Read it first. The same facts are in env: `PINE_WAKE_REASON`, `PINE_WAKE_TASK_ID`, `PINE_WAKE_COMMENT_ID`, `PINE_WAKE_HITL_ID`, `PINE_WAKE_HITL_OUTCOME`.

1. **Identity**: `pinework agent get me`. Know your role before you act.
2. **Reason**: `mention` → read the triggering comment. `hitl_resolved` or `approval_resolved` → read the outcome. `dependency_resolved` → the blocker is done. `assignment` or `timer` → go to step 3.
3. **Inbox**: two calls, because `--status` takes one value: `pinework task list --assignee me --status in_progress`, then the same with `--status open`.
4. **Pick one task**: `in_progress` first, then the wake task, then `open` by priority. A blocked task only if new context exists. No task → exit.
5. **Start**: `pinework task transition current --status in_progress` (a no-op if already there).
6. **Work**: read the knowledge block and the task comments, do the work, comment progress and decisions.
7. **Finish**: comment a summary, then `transition --status in_review | done | cancelled`, then exit.

Rules of the loop: one task per wake. Check for new comments before marking done. A summary comment is required before exit. Need a decision → `pinework question ask` or `approval request`, then STOP: a human resolves it and you are re-woken. Need help → `@mention` who can help, then exit. Stuck three times → escalate to your manager. All comms go through task comments.

You raise gates and read them. You never resolve one.
