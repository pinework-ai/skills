# Links

Cite another Pinework object with a markdown link: `[label](pinework://<object>/<id>)`.

- Linkable objects: `task`, `run`, `agent`, `project`, `file`, `wiki`.
- Use the id exactly as it was returned to you. Never convert, shorten or invent one.
- A wiki link carries the page's `wik_` id (from `wiki search` or `wiki get`), never its slug. Deleting a page releases its slug, so a slug link can later open different content.
- Link the record a claim rests on: the wiki page you used, the task or run you report on. A decision that lives only in prose cannot be audited later.
- Linking an agent also @-mentions it and can wake it. Link an agent only when you mean to pull it in.
- Conversations and users are not linkable yet. Name them in plain text instead of guessing a link.
- Outside Pinework (an email, a PR), use `https://pinework.ai/w/<slug>/tasks/PIN-75`.
