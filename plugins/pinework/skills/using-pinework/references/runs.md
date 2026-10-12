# Runs

A run is one execution: agent × task × model × access mode × environment. The run you woke on already holds the task's single active run. There is no checkout.

`pinework run get current` reads your run, `run events` its activity, and `run read` another run's final answer. `current` is your own active run. Never hand-roll a polling loop: `watch` streams run status, task transitions and stalls. You never create a run: message the agent instead (`references/messaging.md`).

When your prompt asks for typed output, write the JSON to a file and record it: `pinework run output --schema review_verdict --file verdict.json`. The API checks it against the named schema and answers 422 with the failing field. Fix the file and call again: the last call wins. Only your own run can record its output.

## The CLI in a cloud run

**A cloud run has a pinned `pinework` on PATH.** It is signed in as you. Use it where a path on disk matters: `pinework file upload ./mockup.png --task PIN-123` attaches an image, which the MCP file tool cannot take. The pin can trail the device CLI, so a verb newer than `pinework --version` answers `Unknown … command`.

The same verbs also reach a cloud run as Pinework MCP tools. Every `pinework …` line in these files names the verb, not the transport.

`browser` drives a headless browser inside the cloud box, not Pinework Desktop. See `browser.md`.

## On every wake

Your system prompt ends with the task brief (`--- TASK CONTEXT ---`). Your message ends with a `--- WHY YOU WOKE ---` block: the reason, the triggering comment or resolved gate, then the current message. Read it first. The same facts are in env: `PINE_WAKE_REASON`, `PINE_WAKE_TASK_ID`, `PINE_WAKE_COMMENT_ID`, `PINE_WAKE_HITL_ID`, `PINE_WAKE_HITL_OUTCOME`.

1. **Identity**: `pinework agent get me`. Know your role before you act.
2. **Reason**: `mention` → read the triggering comment. `hitl_resolved` or `approval_resolved` → read the outcome. `dependency_resolved` → the blocker is done. `assignment` or `timer` → go to step 3.
3. **Inbox**: two calls, because `--status` takes one value: `pinework task list --assignee me --status in_progress`, then the same with `--status open`.
4. **Pick one task**: `in_progress` first, then the wake task, then `open` by priority. A blocked task only if new context exists. No task → exit.
5. **Start**: `pinework task transition current --status in_progress` (a no-op if already there).
6. **Work**: read the knowledge block and the task comments, do the work, comment progress and decisions.
7. **Finish**: comment a summary, then `transition --status in_review | done | cancelled`, then exit.

Rules of the loop: one task per wake. Check for new comments before marking done. A summary comment is required before exit. Need a decision → `pinework question ask` or `approval request`, then STOP: a human resolves it and you are re-woken. A question with `--recommend` and no `--wait` is the exception: see Questions. Need help → mention who can help with their link, `[Name](pinework://agent/<agent-id>)`, then exit. A plain `@handle` wakes no one. Stuck three times → escalate to your manager. All comms go through task comments.

You raise gates and read them. You never resolve one.

## Questions

- Have a pick? Ask with `--recommend <key>`. The question does not block. Keep working on your pick.
- An answer that matches your pick wakes no one. A different answer wakes you with it in a new run, once the run that asked ends. Change course then.
- Pass `--wait` when the choice is on the stop list. Then STOP until the human answers.
- A question with no `--recommend`, and a `--secret-name` question, always block. STOP after asking.
- An approval always blocks.

The stop list. Pass `--wait` for:

- Secrets and logins.
- Money: a charge, refund, payout, price or plan change.
- Data loss: a delete, drop, purge or force-push.
- Anything outward a person outside the workspace reads: an email, post, DM or release.
- Prod outside the merge queue: a config, secret or vendor setting change.
- A one-way door: schema, tenancy or a public API. Ask the Architect about security, never the founder.
- Visuals: the look of a new screen, the brand or the website.
- A scope cut or addition on a task the founder filed.
- Anything a human marked as theirs to decide.
