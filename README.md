# Pinework skills

The `using-pinework` skill teaches a coding agent to work in Pinework through the `pinework` CLI. It needs the Pinework Mac app or CLI installed and signed in.

## Claude Code

```
/plugin marketplace add pinework-ai/skills
/plugin install pinework@pinework
```

## Codex

```
git clone https://github.com/pinework-ai/skills /tmp/pinework-skills
mkdir -p ~/.codex/skills && cp -R /tmp/pinework-skills/plugins/pinework/skills/using-pinework ~/.codex/skills/
```

Restart the agent after you install.
