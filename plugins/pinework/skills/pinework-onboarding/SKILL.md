---
name: pinework-onboarding
description: >-
  Run the user's first conversation with Chief: learn their work, propose a starting team, create it on a yes, and offer to read their recent Claude Code sessions. Use in the "Getting started" conversation.
---

# pinework-onboarding

The user just finished signing up. Your first message is already posted. It asked what they work on and offered to read their Claude Code sessions. Their reply woke you.

Keep each reply to five lines or fewer. Ask one question at a time. Never show a CLI command, id or error to the user.

## 1. Learn their work

Ask until you know three things: what they build, who it is for, and what takes their time this week. Two or three short questions is enough. Do not interview them.

If they said yes to the sessions read, go to step 4 first. Their sessions answer most of these questions.

## 2. Propose a starting team

Propose 2 to 4 agents. Each has a name and one job in one line. Tie each job to something the user said.

```
Here's a team I'd start with:
- Scout: reads your support inbox and files each bug as a task.
- Forge: fixes the bugs Scout files, one pull request each.
- Quill: writes the weekly changelog from what merged.
Want me to create them? You can rename or drop any.
```

One agent per job. Never propose an agent whose job overlaps another's.

## 3. Create the team on a yes

Create nothing until the user says yes. If they change the list, create the changed list.

Create each agent with the `create-agent` skill. Then tell the user each agent's @handle in one line, and suggest one first task for one agent. File it only on a yes.

## 4. Read their Claude Code sessions

Ask first, and say the path:

```
Can I read your recent Claude Code sessions? They're in ~/.claude/projects on this computer. I'll summarize them and suggest next steps.
```

On a yes, create a task for yourself that runs on their computer:

```
pinework run get current --fields conversationId
pinework task create --assignee me --env device --title "Read and summarize recent Claude Code sessions" --description-file brief.md
```

The brief tells the task's run to:

1. List the folders in `~/.claude/projects`, newest first. Each `.jsonl` file is one session.
2. Read the 10 newest sessions. Skip any file over 5 MB and say so.
3. Write the summary: the projects, what the user worked on, what is unfinished.
4. Propose 2 or 3 next steps. Each is one task and the agent that would own it.
5. Post the summary and the next steps to this conversation with `pinework conversation message post <cnv_…> --body-file <file>`. Keep the post under 150 words.
6. Close the task with the summary as its last comment.

Put the `cnv_` id from the first command in the brief. Tell the user you started, and that the summary lands in this conversation when it is done.

For many sessions, split the reading into several tasks, one per project folder. Each task posts its own summary. The last one proposes the next steps.

If the task does not start, the computer may be offline. Ask the user to open the Pinework desktop app.

## 5. Hand off

When the team exists and one task is filed, say what happens next in one line: the agent works, and the user sees it on the board. Then stop asking questions.
