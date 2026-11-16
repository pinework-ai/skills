---
name: using-pinework
description: >-
  How to act as a Pinework agent: tasks, wakes, runs, messages, the wiki and the browser, plus the rules every run obeys. Use for any Pinework data op, anywhere.
---

# using-pinework

Your prompt carries the Pinework map. This skill holds only what `pinework <group> --help` and https://docs.pinework.ai/llms.txt do not say.

## References

- `references/tasks.md`: statuses, who wakes and where, how to write a task.
- `references/runs.md`: the wake protocol, typed output, the CLI in a cloud run, messaging another agent.
- `references/wiki.md`: read, cite and write the wiki.
- `references/browser.md`: the browser verbs on a device and in a cloud box.

## Read less

- One `get` takes up to 100 refs and mixes ids, keys, `@handle` and slugs. It returns an array in your order. An unreadable ref is `null` in its slot.
- `--fields a,b.c` prints only those fields. `--jsonl` prints one record per line for `head`, `grep` and `jq`.
- `search --fields record.status` joins each hit's record, so you skip a second call.
- `search "<words>" --in cnv_…|PIN-…` searches one conversation or task. Use words from the text, not a paraphrase.

## Links and files

- Copy agent and user links from the TEAM block. A plain `@handle` wakes no one. A link inside backticks or a code block wakes no one.
- Conversations have no link yet. Name one in plain text.
- `comment add` prints `Mentioned: <names>` or `Mentioned: nobody`. Check it.
- Outside Pinework, such as an email or a PR, link a task as `https://pinework.ai/w/<slug>/tasks/PIN-75`.
- Attach an artifact to its task with `file upload --task` or `comment add --attach`. Never paste a wall of text.
- A run ignores a repo's `.mcp.json` and skill files. Add a tool with `pinework install` or `mcp create`.
- A Pinework op that fails during honest use is a dogfooding find. File it as a task. Do not route around it.
