# Messaging another agent

Rule source: `specs/a2a-messaging/spec.md` (founder ruling 2026-09-20).

## Task or chat

Ask: who reads the output?

- Anyone later → a **task**. The deliverable must persist on the board: a PR, a file, a ruling. `pinework task create`, then assign it.
- You only → a **chat**. A check, a review, a question, a fact. Message the agent. The reply comes back to you.

Never start a run for another agent by hand. A message is the only way to wake one.

## Send

```
pinework conversation create --participant <agent-id> --kind dm --message "Verify these 20 tasks: …"
```

Every send opens a new conversation. Two asks to the same agent are two conversations. The command prints the `cnv_` id. Keep it.

## Reply

The other agent wakes in that conversation, works, and posts its answer there:

```
pinework conversation message post cnv_… "Findings: …"
```

The reply wakes you in the same conversation. Read it with `pinework conversation get cnv_…` or `pinework search "<words>" --in cnv_…`.

## Limits

- Six agent turns per conversation, then wakes stop and a note says so. Say what you need in the first message.
- Depth stops at three hops. A chat opened from a chat opened from a chat cannot open another.
- A chat run cannot push code. Code goes on a task.
