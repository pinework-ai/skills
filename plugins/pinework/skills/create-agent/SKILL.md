---
name: create-agent
description: >-
  Create an agent in the user's workspace with the CLI, and write its instructions. Use after the user agrees to a new agent.
---

# create-agent

Create an agent only after the user says yes to its name and its job.

## 1. Write the instructions to a file

The instructions are the agent's standing guidance. Every run reads them. Write them as laws, in this shape:

```
# Scout

You are Scout. You read the support inbox and file each bug as a task. You fix nothing.

## You own
- The bug queue: one task per bug, with steps to reproduce.

## You refuse
- Fixing a bug. File it for Forge.
- Filing a duplicate. Search the open tasks first.

## Laws
- Every bug task names where the report came from.
- Ask the user before you close a report as "not a bug".
```

A good instruction:

- Opens with who the agent is and its one job, in two sentences.
- Names what the agent owns and what it refuses. The refusals keep two agents from doing the same work.
- Gives laws, not advice. "Search the open tasks first", not "try to avoid duplicates".
- Uses the user's own words for their product, their tools and their customers.
- Stays under 300 words. Every run pays for every word.

Leave out tone guides, long backstory and CLI flags. The agent already knows how to use Pinework.

## 2. Create the agent

```
pinework agent create --name "Scout" --role "Support triage" --description "Reads the support inbox and files each bug as a task." --agent-md scout.md
```

- `--name` sets the name. Pinework derives the @handle from it.
- `--role` is a short label the user sees next to the name.
- `--description` is one line on what the agent does.
- `--agent-md` uploads the instructions file.

Leave `--model` out. The agent then uses the workspace's default model, and the user can change it later in agent settings.

## 3. Check it

The create output carries the agent's `handle`. Read the agent back with it:

```
pinework agent get <handle> --fields name,handle,role
```

Tell the user the agent's name and @handle.

Run `pinework agent create --help` for every flag.
