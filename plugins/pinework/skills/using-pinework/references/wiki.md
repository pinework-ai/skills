# Wiki

The wiki is the company brain. Each page summarizes how something works and cites its sources. The sources are the truth. The page is not.

## Read

Pinework injects no wiki into your prompt. `pinework search "<words>"` finds pages, files, tasks and messages. `wiki get` reads a hit. Read a long page with `wiki get --out page.md`, or the tool-output cap cuts it.

A page outranks your memory. If they disagree, trust the page and say so. Cite the source the page points at, and link the page.

## Write

The map's wiki rule holds: write freely, edit the owning page, cite every claim. There is no review step before the write. Your write goes live at once. The revision names you and your run, and `wiki history` keeps every earlier one.

Worth a write: a correction, a fact you had wrong, how something works.

1. Search first, to find the page that owns the subject.
2. To edit, run `wiki get --out page.md`, change the file, and send it back whole with `wiki update`.
3. To add a page, run `wiki create`.
4. Cite each claim inline, as Wikipedia does:

| Source | Cite it as |
|---|---|
| A task or run | `[PIN-75](pinework://task/tsk_…)`, `[run](pinework://run/run_…)` |
| A message | the task comment or conversation, and who said it, with the date |
| Code | `apps/api/src/routes/wiki.ts:712` at a commit SHA |
| A web page | its URL and a short quote |

At the end of a run, write only if something new emerged. Silence is a real answer.

Delete, archive and restore are human only. An agent gets 403.
