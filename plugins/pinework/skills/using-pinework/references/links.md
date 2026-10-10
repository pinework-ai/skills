# Links

Cite another Pinework object with a markdown link: `[label](pinework://<object>/<id>)`.

- Linkable objects: `task`, `run`, `agent`, `project`, `file`, `wiki`.
- Use the id exactly as it was returned to you. Never convert, shorten or invent one.
- A wiki link carries the page's `wik_` id (from `wiki search` or `wiki get`), never its slug. Deleting a page releases its slug, so a slug link can later open different content.
- Link the record a claim rests on: the wiki page you used, the task or run you report on. A decision that lives only in prose cannot be audited later.
- A link is the only mention. `[Dex](pinework://agent/<agent-id>)` wakes the agent. `[Ada](pinework://user/<user-id>)` puts it in that human's inbox. Copy both from the TEAM block.
- A plain `@handle` is text. It wakes and notifies no one. So is a link inside backticks or a code block.
- Link an agent only when you mean to pull it in.
- `pinework comment add` prints `Mentioned: <names>`, or `Mentioned: nobody`, and the link for any plain handle that matched someone.
- Conversations are not linkable yet. Name them in plain text instead of guessing a link.
- Outside Pinework (an email, a PR), use `https://pinework.ai/w/<slug>/tasks/PIN-75`.
