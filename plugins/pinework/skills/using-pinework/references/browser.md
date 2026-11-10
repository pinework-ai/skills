# The browser verbs

The verbs drive one browser tab. On a device it is the tab inside Pinework Desktop, which runs on macOS only. In a cloud run it is a headless browser in the box, and nobody watches it.

- Refs come from `snapshot`. Never guess `@e4`. Snapshot again after the page changes, or the verb fails `stale_ref`.
- On a device, any verb starts Desktop hidden if it is not running. `app_unavailable` means this is not a Mac, Desktop is not installed, or it is too old. The error says which. Do not retry. Report what you found.
- On a device, the tab is the human's. Comment the URL and what you clicked. Paste the snapshot when it is the evidence.
- These verbs are not `agent-browser`, which drives a Chromium nobody can watch.

## `fill`: a vaulted login

`pinework browser fill <login>` types a login from the vault into the open page. The login must be granted to you. The page's origin must be one of the login's URLs.

- You never see the value. The verb prints only which boxes it filled.
- A two-step login fills twice. Submit the username, then run `fill` again on the next page.
- On a device, a Desktop too old for `fill` answers `unknown_verb`.

## `exec`: when `snapshot` cannot see the control

`snapshot` misses a `div` with a click handler, a closed shadow root, a cross-origin frame and some banners. Write a script instead.

```bash
pinework browser exec --js "const el = [...document.querySelectorAll('a,button')].find(e => /sign in/i.test(e.textContent)); if (!el) throw new Error('no match'); el.click(); el.textContent.trim()"
pinework browser exec --cdp Page.captureScreenshot --params '{"format":"png"}' --out "${TMPDIR:-/tmp}/page.png"
```

- The last expression is the answer. Top-level `return` is a syntax error. Wrap an object literal in parentheses.
- Top-level `await` works. Use it to wait for a selector.
- Return the few fields you need, never a DOM node.
- Use `--js-file` for a long script. Shell quoting breaks scripts.
- A script that throws exits non-zero, so `&&` chains stop.
- The panel shows the first 80 characters of your script and never its result. Say in a comment what came back.
- A script can read a password a human typed into the tab. Never read a field you were not sent for. Never paste one.
