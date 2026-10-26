# Read what the coding agents know

Use this when the user says yes to reading this Mac.

## 1. Say what you will read

```
I'll read your Claude Code and Codex memory, the CLAUDE.md, AGENTS.md and Cursor rules in your projects, and your own messages from the last 30 days. Secrets are removed first. Nothing leaves this Mac until you say yes.
```

If they use Cursor's memory export, ask them to export it and give you the file path.

## 2. Run the reader

```
pinework import knowledge
```

With a Cursor export: add `--cursor-export <path>`. With an Agent Memory Repo folder: add `--amr <path>`.

Run it as a plain command. No `$(…)` and no backticks in it.

It prints JSON:

- `path`: the bundle file on this Mac.
- `sources`: how many memory, rules and transcript sections it found.
- `chars`: the bundle size.
- `redactions.total`: how many secrets it removed.
- `unreadable`: folders macOS would not let it read, each with its error code. Usually empty.

## 3. Read the bundle

Read the file at `path` with your Read tool. Read it all before you propose anything.

The kinds of section:

- **Memory**: facts the coding agents saved on their own. The richest source.
- **Agent Memory Repo**: one bullet is one fact. Its `source:` link is that fact's source.
- **Cursor memory export**: facts Cursor saved.
- **Project rules**: CLAUDE.md, AGENTS.md and `.cursor/rules` files. Use them only to learn what each project is and how it is built. Never copy their instructions as facts. `pinework import apply` brings rules in (config.md).
- **Transcript user turns**: the user's own words. Most are one-off asks. Keep only what lasts: a preference, a decision, a correction ("never do X", "always Y"), who they are, what a project is for.

## 4. Tell the user

One line: how many sources you read and how many secrets you removed. Then go to the proposal.

If every count in `sources` is 0, say you found nothing on this Mac. Offer the paste step, or ask what they work on.

If `unreadable` lists folders, name them. Tell the user to allow Pinework in System Settings, Privacy & Security, Files and Folders, then offer to run it again.
