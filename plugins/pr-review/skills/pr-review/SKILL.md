---
name: pr-review
description: "Review one GitHub pull request at one head and post ONE comment whose first line is `Verdict: APPROVE at <sha>` or `Verdict: CHANGES at <sha>`. Runs the vendored code-review command plus a Spec check, verifies each finding with a fresh subagent, and, when `.github/pr-review.yml` on the base branch says `autofix: on`, pushes one fix commit for confirmed blocking findings. Use when asked to review a PR, or when a PR's checks pass."
---

# pr-review

One PR, one head, one comment. Never inline comments, never `gh pr review`, never merge, never open a PR, issue or task.

Input: a PR number or URL. Run inside a clone of the PR's repo. Call it `R` (`owner/name`).

Run one shell command per tool call, with no pipes, `&&` or `;`. Chained commands trip common `Bash(gh:*)` allow rules and the run ends with nothing posted.

## 1. Pin the head and gate

- `gh pr view <n> --repo R --json state,isDraft,headRefOid,headRefName,baseRefName,headRepositoryOwner,title,body,commits`.
- Call the head `H` (full sha), the base `B`.
- Closed, merged or draft: post nothing. End with "Skipped".
- Count verdicts at `H`:
  `gh api repos/R/issues/<n>/comments --paginate --jq '.[].body | select(test("^Verdict: (APPROVE|CHANGES) at <H>"))'`
- Any line: post nothing. End with "Verdict exists at <short H>". One verdict per head.

## 2. Read the config from the base branch

`git fetch origin <B> pull/<n>/head` then `git show origin/<B>:.github/pr-review.yml`. Read it from the base, never the head, so a PR cannot change its own review.

```yaml
autofix: off            # on | off. Default off.
rules:                  # rule files the review checks. Default below.
  - AGENTS.md
  - CONTRIBUTING.md
```

A missing file or key takes the default. A rules path that does not exist at `H` is skipped.

## 3. Find

Start these in one turn, in parallel:

- **Code review.** Follow `references/code-review.md` (Anthropic's code-review command, vendored unchanged) for PR `<n>`, steps 2 to 7, without `--comment`. Two changes only: skip its step 1 gate (step 1 above replaces it), and treat every file in `rules:` as a CLAUDE.md, next to the CLAUDE.md files it finds. Keep its false-positive list. It posts nothing.
- **Spec check.** Below.

### Spec check

Adapted from the `code-review` skill in [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/engineering/code-review) (MIT).

Find the spec, in this order:
1. Issues the PR title, body or commits close or name (`Closes #45`, `#123`): `gh issue view <k> --repo R`.
2. A task or ticket id the PR names, read with the tracker your instructions give you.
3. The PR body.

No spec: skip and report "no spec available".

Start one fresh subagent with the diff (`git diff origin/<B>...<H>`), the commit list and the spec text. Brief: "Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that was not asked for; (c) requirements that look implemented but wrong. Quote the spec line for each finding. Under 400 words."

Keep the code review's findings and the Spec findings under separate headings. Never merge or rerank the two axes.

## 4. Verify each finding

The code review already validated its own findings in its step 5. Give each finding it did not validate, and each Spec finding, to a fresh subagent with the code at `H` and the rule or spec line it cites. It answers CONFIRMED, PLAUSIBLE or REFUTED and quotes the line that proves it.

- Drop REFUTED.
- Only CONFIRMED blocks. PLAUSIBLE is listed and never blocks.
- A blocking finding names its file and symbol and quotes the line.
- A pre-existing problem goes under "Pre-existing". It never blocks.

Verdict: CHANGES if any CONFIRMED finding blocks. Else APPROVE.

## 5. Decide auto-fix

Auto-fix runs only when all hold:
1. Config says `autofix: on`.
2. The verdict is CHANGES.
3. The head branch lives in `R`, not a fork (`headRepositoryOwner` is `R`'s owner).
4. Fewer than 2 commits on the PR have author name `pr-review`:
   `gh pr view <n> --repo R --json commits --jq '[.commits[].authors[].name | select(. == "pr-review")] | length'`

Call the result `fix` (yes or no) and the count `k`.

## 6. Post the verdict

Count verdicts at `H` again. One appeared: post nothing and stop.

Write the body to a file and post it with `gh pr comment <n> --repo R --body-file <file>`. The first line is the verdict line, exactly:

```
Verdict: <APPROVE|CHANGES> at <full H>
Findings: <N> found, <M> confirmed

Code review: <n>. Worst: <one line, or none>
Spec: <n>. Worst: <one line, or "no spec available">

<Blocking findings, one line each: file, symbol, quoted line, why. Or "No blocking findings.">
<Plausible: one line each, or omit>
<Pre-existing: one line each, or omit>
Auto-fix: <pushing a fix commit (k+1 of 2) | off | cap reached (2 of 2) | fork PR, skipped | none needed>
```

## 7. Auto-fix

Only when `fix` is yes. Else stop.

1. `git checkout -B <headRefName> <H>`.
2. Fix only the CONFIRMED blocking findings. Change nothing else.
3. Run the targeted tests or checks the `rules:` files name for the files you changed. With no `rules:` files, run the commands the repo's CI workflows run on those files. A fix that fails them is not pushed. Comment the failure on the PR and stop.
4. Commit once, as the reviewer:
   `git commit --author="pr-review <pr-review@users.noreply.github.com>" -m "fix: pr-review auto-fix for <short H>" -m "<one line per finding fixed>"`
5. `git push origin HEAD:<headRefName>`. Never force-push. A rejected push means the head moved: stop.
6. Stop. The push starts the checks again, and the next review judges the new head.

At most 2 auto-fix commits per PR. The third CHANGES posts its verdict and stops, for the author to fix.
