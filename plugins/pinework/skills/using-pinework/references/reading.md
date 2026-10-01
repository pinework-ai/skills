# Reading less

Ask for the fields you need, ask for every record at once, pipe the rest through bash.

## Read many records in ONE call

Every `get` verb takes several refs: `task`, `agent`, `file`, `run`, `wiki`, `conversation`. Reading 18 tasks is one request, not 18.

```
pinework task get PIN-3434 PIN-3435 tsk_9i0S17FUVOUNt3Sc --fields key,status,assignee.id
pinework agent get @ash @iris --fields handle,role
pinework wiki get device-runtime run-lifecycle --fields title,updatedAt
```

- Ids and keys mix freely. `PIN-42` and `tsk_…` in the same call, `@handle` for agents, a slug for a wiki page.
- Output is an ARRAY in the order you asked, one slot per ref. A ref you cannot read is `null` in its slot, so position `i` always answers ref `i`. Nothing is dropped.
- One ref still prints the object, not a one-element array.
- Cap is 100 refs. `--fields` and `--include` work on the batch exactly as on one record.
- If the answer does not have one slot per id, the CLI refuses to print it rather than showing you a page you did not ask for.
- `ids` cannot combine with a filter: `task get` many refs is "these exact records", so `--status` there is a 400, not a silent no-op.

## Ask for fewer fields

- `--fields a,b.c` prints only those fields. A dot path reaches inside (`assignee.id`). A field on no record is an error that names the fields present, never an empty record.
- `--jsonl` prints one record per line, no envelope. A list prints its rows, a get prints its one object. When more rows exist, the next cursor goes to stderr, so stdout stays clean for `head`, `grep`, `jq -c`.
- `search --fields record.status` joins each hit's SOURCE record, so you can read a task's status or a file's contentType without a second call. Only `record.*` paths turn the join on, so a plain search stays as cheap as before.
- `search "<words>" --in cnv_…|PIN-…|tsk_…` greps one conversation or task: its messages and attached text files, newest first, every word must match. `--kind conversation|file` narrows, `--top-k` caps (10 default, 50 max). Without `--in`, `search` is the workspace index: wiki pages, files and tasks in one list by score.

## One example per verb family

```
pinework task get PIN-75 --fields key,status,assignee.id
pinework task list --status open --jsonl --fields key,title | head -20
pinework comment list PIN-75 --jsonl --fields author.name,body | grep -i shipped
pinework run list --task PIN-75 --jsonl --fields id,status | jq -c 'select(.status=="failed")'
pinework file list --task PIN-75 --jsonl --fields id,name,contentType
pinework search "launch video" --in cnv_…              # what was said, one conversation
pinework search "budget" --kind file --in PIN-75        # text inside the task's files
pinework agent list --jsonl --fields handle,status
pinework search "lease renewal" --kind task --fields sourceId,title,record.status
```

Measured on `task get` and `task list --limit 25`, 2026-09-18: 1,365 → 27 tokens and 6,499 → 846 tokens with `--fields key,title,status`. The full record is one flag away when you need it.

On the API the same thing is `GET /v1/<primitive>?ids=a,b,c` — tasks, agents, files, runs, wiki pages and conversations. Same cap, same order, same null slots.
