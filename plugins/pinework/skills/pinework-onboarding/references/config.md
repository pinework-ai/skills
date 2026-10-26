# Set up plugins, tools and GitHub

Each step asks first. Skip a step the user does not want. The links below use the workspace slug from the dashboard URL, shown here as `<slug>`.

## Plugins, skills, MCP servers and hooks from this Mac

Ask: "Want me to bring the plugins, skills and MCP servers you use in Claude Code?"

```
pinework import scan --source claude-code
```

Use `--source codex`, `opencode` or `cursor` for the other tools. The output tags each item `new`, `changed` or `same`. Tell the user what is new, by kind, in a few lines. Ask which to keep. Then:

```
pinework import apply --only skill:<name> --only mcp:<name>
pinework import apply --all-new
```

A secret the user does not pick never leaves the Mac. An MCP server that needs a sign-in shows **Not connected** until the user connects it at `https://pinework.ai/w/<slug>/customize/mcp-servers`.

Settings page, if a command fails: `https://pinework.ai/w/<slug>/customize/import`.

## Tools

Ask which tools they use, such as Notion, Linear or Gmail. Then ask which agents should use each one. A connection reaches no agent until you name it. Get the agent ids from `pinework agent list`. For each tool:

```
pinework connector search <tool name>
pinework connection create --integration-id <id> --agent-ids <agent-id,agent-id>
```

The output's `nextStep` says what comes next:

- `oauth_redirect`: open its `authorizeUrl` for the user with `pinework browser open <authorizeUrl>`. In a browser session, send the link instead. The user signs in there.
- `api_key_fields`: never ask for a key in the chat. Send the link `https://pinework.ai/w/<slug>/customize/connectors` and name the tool.
- `none`: it is connected.

## GitHub

Ask: "Should your agents work on your GitHub repos?"

```
pinework github connect
```

It prints the GitHub App install URL. Open it with `pinework browser open <url>`, or send the link. The user picks the repos there.

Settings page: `https://pinework.ai/w/<slug>/settings/repos`.

## Your AI model

Onboarding already made sure a model is ready, or you could not run. To add a second subscription or an API key, send `https://pinework.ai/w/<slug>/settings/providers`.
