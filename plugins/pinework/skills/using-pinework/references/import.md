# Import

`pinework import` brings a harness config on this device into the workspace: skills, plugins, MCP servers, hooks and secrets. Two steps. Nothing uploads until you name it.

`pinework import scan`, then `pinework import apply`. `pinework import <cmd> --help` has the flags.

`scan` reads the local config and prints every item, tagged against the workspace:

- `new`: not in the workspace.
- `changed`: in the workspace, content hash differs (skills carry a bundle hash, MCP servers a config hash, hooks their command, plugins their version and child skills).
- `same`: already there as-is. A secret is never `changed`: the vault value is opaque, so an existing name is `same`.

JSON by default, one object per item: `{kind, name, status, detail}`. `--pretty` prints a table. The scan sends identity only: slugs, names, hashes, env key names. No skill body, no secret value.

`apply` uploads only what is named. Kinds: `skill` (slug), `plugin` (slug), `mcp` (server name), `hook` (event name, every hook on it), `secret` (vault name). `--only` repeats; `--all-new` and `--all-changed` combine with it. New items create, changed items update in place (a skill's bundle is replaced, a hook's command is replaced and the hook is turned off again), same items are skipped and named. No selector: nothing uploads, non-zero exit. A selector that matches nothing fails the run and prints what the scan found. A secret you did not select never leaves the machine: its MCP env entry becomes a `vault:<name>` reference you can fill with `pinework secret create`.

Rules and settings do not import from the CLI. Permissions: `pinework rules add`. Instruction files: the dashboard import page.
