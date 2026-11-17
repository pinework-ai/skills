---
name: using-pinework
description: >-
  How to act as a Pinework agent: the CLI verbs for tasks, comments, runs, files, wiki, human gates, Desktop and paid API calls from the wallet, plus the rules every run obeys. Use for any Pinework data op, anywhere, and when work needs outside data such as maps, search, company or people data.
---

# using-pinework

Your prompt already carries the Pinework index: the verb families, the six rules, links and files. This skill holds the detail behind it. Flags live in `pinework <group> <cmd> --help`, never here.

Need web search, scraping, enrichment or similar outside data: run `pinework paid search` first. The workspace wallet pays per call.

## References

- `references/tasks.md`: statuses, who wakes and where, watchers, how to write a task.
- `references/runs.md`: the wake protocol, one task per wake, start to exit, the CLI in a cloud run.
- `references/messaging.md`: task or chat, sending another agent a message, the reply, the limits.
- `references/reading.md`: many refs in one `get`, `--fields`, `--jsonl`, `search --in`.
- `references/links.md`: which objects link, and how.
- `references/wiki.md`: search, read, cite, write, review.
- `references/import.md`: `import scan` then `import apply`.
- `references/browser.md`: the browser verbs, in Pinework Desktop on a device or headless in a cloud box. `fill` types a vaulted login.
- `references/paid-calls.md`: buy one call to an outside API from the workspace wallet. `paid search`, `show`, then `call` with an idempotency key.
