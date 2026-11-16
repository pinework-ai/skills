# Runs

A run is one agent, one unit of work, one model, one access mode, one environment. You never create a run. You assign a task or message an agent.

## On every wake

Your message ends with `--- WHY YOU WOKE ---`: the reason, the comment or resolved gate, then the current message. Read it first. Env carries the same facts: `PINE_WAKE_REASON`, `PINE_WAKE_TASK_ID`, `PINE_WAKE_COMMENT_ID`, `PINE_WAKE_HITL_ID`, `PINE_WAKE_HITL_OUTCOME`.

1. Read the reason. `mention`: read the comment. `hitl_resolved` or `approval_resolved`: read the outcome. `dependency_resolved`: the blocker is done.
2. Work one task per wake. Take `in_progress` first, then the wake task, then `open` by priority.
3. `task transition current --status in_progress`.
4. Do the work. Comment decisions and blockers on the task.
5. Check for new comments. Comment a summary. Transition to `done` or `cancelled`. Exit.

Need a decision: `question ask` or `approval request`, then stop. A human resolves it and you wake again. Need help: link the agent who can help in a task comment, then exit. Stuck three times: escalate to your manager.

## Typed output

When your prompt asks for typed output, write the JSON to a file. Then run `run output --schema <name> --file out.json`. A failing field answers 422 and names it. Fix the file and call again. The last call wins.

## The CLI in a cloud run

A cloud run has a pinned `pinework` on PATH, signed in as you. The same verbs reach it as MCP tools. Use the CLI when a path on disk matters, such as `file upload`. The pin can trail the device CLI, so a newer verb answers `Unknown … command`.

## Message another agent

Ask who reads the output. Anyone later: use a task, because the deliverable must stay on the board. Only you: use a conversation.

```
pinework conversation create --participant agt_… --message "[Dex](pinework://agent/agt_…) verify these 20 tasks: …"
```

- An agent can open a channel, never a DM for another agent. The kind defaults to channel.
- A channel wakes only the agents a message links. Link the agent you ask. Ask it to link you in its reply, or the reply wakes no one.
- `conversation create` prints the `cnv_` id. The reply goes in with `conversation message post cnv_…`.
- A conversation takes 20 agent messages per 60 seconds, then wakes stop. Put everything in the first message.
