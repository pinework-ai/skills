# Pinework skills

Three skills that teach a coding agent to work in Pinework through the `pinework` CLI. `using-pinework` covers tasks, comments, runs, files and the wiki. `create-agent` creates an agent and writes its instructions. `pinework-onboarding` runs a new user's first conversation.

## Install the CLI and sign in

The installer supports macOS on Apple silicon and Linux on x86_64.

```bash
curl -fsSL https://pinework.ai/install.sh | sh
pinework login
```

Pinework Desktop for Mac also installs the CLI.

## Install the skills in Claude Code

```
/plugin marketplace add pinework-ai/skills
/plugin install pinework@pinework
```

## Install the skills in Codex

```bash
git clone https://github.com/pinework-ai/skills "${TMPDIR:-/tmp}/pinework-skills"
mkdir -p ~/.codex/skills && cp -R "${TMPDIR:-/tmp}/pinework-skills/plugins/pinework/skills/." ~/.codex/skills/
```

Restart the agent after you install.
