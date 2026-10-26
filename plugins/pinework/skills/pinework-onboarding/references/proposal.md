# Propose once

One message holds everything: the agents, the wiki pages, and what you dropped. The user answers in words.

## Rules

1. Group the work you see into jobs, such as "builder for the shopfront repo", "code reviewer" or "researcher". Name agents by job. Never one agent per project by default. One project often needs a builder and a reviewer.
2. Propose about 3 to 5 agents. Propose more or fewer when the facts call for it. Drop an agent when too few facts back it, and say so under Dropped.
3. One agent per job. Never two agents whose jobs overlap.
4. For each agent give: a short name, its job in one line, and one line of why, tied to something in the source.
5. Facts every agent needs go on a wiki page, never into every agent:
   - one "About <user>" page: who they are, how they like work done, their corrections.
   - one page per project: what it is, who it is for, the stack, how it is built and tested.
   - other pages only when needed.
6. Each fact appears once. One plain sentence, present tense. End it with its source in brackets: the `Source:` path from the bundle section, the `source:` link of an Agent Memory Repo bullet, or `[paste]`.
7. List what you did not use under Dropped, with one reason. Group near-identical drops on one line.
   - duplicate: the same as a kept fact.
   - stale: a newer fact replaces it, or its date shows it no longer holds. Name what replaced it.
   - personal: private life with no bearing on work.
   - thin: too little to act on, or a one-off ask.
8. End with one question: what to change, or yes to save it all.

## Shape

```
Here's what I'd set up.

**Agents**
- Forge: builds the shopfront storefront. You ship most of your work there.
- Lint: reviews every pull request before you merge. You review each PR yourself today.
- Scout: researches vendors and pricing. You asked for a Stripe vs Wise comparison.

**Wiki pages**
- About Sam: 6 facts
- shopfront: 9 facts
- ledger-api: 5 facts

**Dropped**: 4 personal, 3 one-off asks, 1 stale (ledger-api moved from Heroku to Fly.io).

Want me to save it all? Or tell me what to change.
```

Show the facts of a page only when the user asks. Keep the full list, with sources, for the save.
