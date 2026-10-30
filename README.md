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

## pr-review: a PR reviewer with optional auto-fix

`pr-review` reviews one pull request at one head and posts one comment. Its first line is `Verdict: APPROVE at <sha>` or `Verdict: CHANGES at <sha>`. It never posts inline comments.

- It runs Anthropic's [code-review](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-review) command, vendored unchanged (Apache-2.0, see `plugins/pr-review/NOTICE`).
- It adds a Spec check adapted from [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/code-review) (MIT): does the diff do what the issue or PR asked?
- A fresh subagent checks each finding. Only confirmed findings block.
- With auto-fix on, it pushes one fix commit to the PR branch for confirmed blocking findings. It stops after 2 such commits per PR. It never opens a new PR, issue or task, and skips fork PRs.

### Configure it

Add `.github/pr-review.yml` to your default branch. The reviewer reads it from the PR's base branch, so a PR cannot change its own review.

```yaml
autofix: off        # on | off. Default off.
rules:              # files the review checks the diff against
  - AGENTS.md
  - CONTRIBUTING.md
```

### Install in Claude Code

```
/plugin marketplace add pinework-ai/skills
/plugin install pr-review@pinework
```

Then ask: "review PR 123 with pr-review".

### Set it up in Pinework

1. Add this marketplace once, then install the plugin for your reviewer agent:
   `pinework marketplace add https://github.com/pinework-ai/skills`
   `pinework plugin install --marketplace pinework --plugin pr-review --agent <reviewer agent>`
2. Give the reviewer agent its job in its instructions: review one PR with the `pr-review` skill, post one verdict, never merge.
3. Create the automation: `pinework automation create --name "PR review" --event pr-checks-passed --repo <owner/name> --agent <reviewer agent> --prompt "Use the pr-review skill on the PR in this event."`. It fires when a task's pull request goes green at a new head.
