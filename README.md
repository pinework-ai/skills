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

### Run it in GitHub Actions

This workflow uses [anthropics/claude-code-action](https://github.com/anthropics/claude-code-action) and runs after your CI workflow passes. Set `workflows:` to your CI workflow's name and add an `ANTHROPIC_API_KEY` secret. Run it only on repos you trust: it runs with your secrets.

```yaml
name: pr-review
on:
  workflow_run:
    workflows: [CI]
    types: [completed]

permissions:
  contents: write        # auto-fix pushes a commit. Use read with autofix off.
  pull-requests: write
  issues: read
  id-token: write

jobs:
  review:
    if: >-
      github.event.workflow_run.conclusion == 'success' &&
      github.event.workflow_run.event == 'pull_request' &&
      github.event.workflow_run.pull_requests[0]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          plugin_marketplaces: https://github.com/pinework-ai/skills.git
          plugins: pr-review@pinework
          prompt: >-
            Use the pr-review skill on PR
            ${{ github.event.workflow_run.pull_requests[0].number }}
            in ${{ github.repository }}.
          claude_args: >-
            --allowedTools "Bash(gh:*),Bash(git:*),Read,Edit,Write,Grep,Glob,Task"
```

To block merges on a CHANGES verdict, add a required check that reads the newest `Verdict:` line at the PR head.

### Set it up in Pinework

1. Add this marketplace once, then install the plugin for your reviewer agent:
   `pinework marketplace add https://github.com/pinework-ai/skills`
   `pinework plugin install --marketplace pinework --plugin pr-review --agent <reviewer agent>`
2. Give the reviewer agent its job in its instructions: review one PR with the `pr-review` skill, post one verdict, never merge.
3. Create the automation: `pinework automation create --name "PR review" --event pr-checks-passed --repo <owner/name> --agent <reviewer agent> --prompt "Use the pr-review skill on the PR in this event."`. It fires when a task's pull request goes green at a new head.
