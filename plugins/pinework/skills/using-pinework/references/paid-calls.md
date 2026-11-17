# Paid API calls

A run can buy one call to an outside API and get its answer back. The workspace wallet pays. The catalog holds maps and places, web search, company and people data, enrichment and more. Prices are cents per call.

## When to use it

Use it when your work needs data or an action you cannot get for free from the repo, the workspace or a plain web fetch. Examples: geocode an address, find local businesses, look up a company or a person, enrich a lead list. Check `paid search` before you say you lack a data source.

Skip it when a free source already answers, or when the result does not change what you do next.

## Search, then show, then call

```bash
pinework paid search "company data" --limit 5
pinework paid show <service-slug>
pinework paid call <service-slug> --input '{"field":"value"}' --idempotency-key <key>
```

1. `search` lists services by words. Each has a `slug`, a `priceUsd` per call and a `maxChargeUsd`. Pick one with `payable: true`.
2. `show <slug>` prints the input fields. Each field says if it is `required` and where it goes: `in: body` goes in `--input`, `in: query` goes in `--query`. Never guess a field name.
3. `call <slug>` pays and runs the call. The answer is in `output`. `chargedUsd` is what the wallet paid. `id` is the call id. Cite it with the answer.

## Idempotency key

Always pass `--idempotency-key`. Make it unique to this one call and stable across retries, such as `<task-number>-<what>-<n>`: `PIN-123-geocode-1`. Letters, digits, `.`, `_` and `-` only, at most 100.

- A retry with the same key and input pays once. Once that call has finished, the retry answers 409 `idempotency_key_used`: the first call's result is the answer, so never retry it with a new key.
- Without a key, the API makes a new one, so a retry pays again.
- `status: pending` means the outcome is not known yet. Retry with the same key. Never with a new one.
- A 409 `idempotency_key_reused` means you used the key for a different call. Pick a new key.

## The wallet

- The wallet holds `maxChargeUsd` before the call and pays only for a call that succeeds. A failed call costs nothing.
- `--max-charge <usd>` sets the most one call may cost. Below the service's max charge, the API refuses it with `max_charge_too_low`.
- A 402 means the wallet or the monthly spend cap has no room. Nothing was called. Do not retry. Say so on your task. `pinework billing wallet` shows the balance.
- Only an agent in a live run can pay. A 403 `run_required` means you are not in one.
- Spend only what the task needs. Say on the task what you bought and what it cost.
