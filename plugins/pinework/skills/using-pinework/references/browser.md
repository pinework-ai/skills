# The browser verbs

A run can look at a page and click through a UI. Six verbs drive one browser tab. On a device run it is the tab inside Pinework Desktop. In a cloud run it is a headless browser inside the cloud box. `browser open <url>` opens or reuses the tab. `navigate <url>` moves the open tab. `snapshot` prints the page as refs you can act on. `click <@eN>` presses the element a ref names. `type <@eN> "text"` types into it. `exec` runs your own script in the page. A seventh verb, `fill <login>`, works in a cloud run only. `pinework browser <cmd> --help` has the flags.

Rules:

- **Refs come from `snapshot`.** Never guess `@e4`. Re-snapshot after the page changes, or the verb fails `stale_ref` and tells you to snapshot again.
- **On a device, any verb starts Desktop hidden if it is not running**, then retries once. No window opens and nothing steals focus. `app_unavailable` means one of three things: this machine is not a Mac, the app is not installed, or the installed build is older than the browser verbs. The error says which, and names the installed version. Do not retry: read the advice, then say what you found.
- **On a device, the tab is the human's.** It is the same tab the right panel shows, not a private browser. So say what you did: post a comment naming the url and what you clicked, and paste the snapshot when it is the evidence.

This is not `agent-browser`. That drives a separate Chromium nobody can watch. These verbs drive the tab a human can open and see.

**Where the tab lives.** On a device it lives in Pinework Desktop, which runs on macOS only. Install or update Desktop when a verb answers `app_unavailable`. In a cloud run the tab is a headless browser in the box, and no human watches it. There is nothing to install, so never send a cloud run to Desktop.

## `fill` — a vaulted login, in a cloud run

`pinework browser fill <login>` types a login from the vault into the open page. Name the login by its id or name. It must be granted to you, and the page's origin must be one of the login's URLs. The API refuses any other page.

```bash
pinework browser open https://github.com/login
pinework browser fill github-login
pinework browser snapshot
```

- **You never see the value.** The verb prints which boxes it filled, `username filled, password filled`, or booleans with `--json`. The value goes from the API to the page and nowhere else.
- **Two-step logins fill twice.** When the page has no password box yet, the verb fills the username and says so. Submit it, then run `fill` again on the next page.
- **On a device it answers `not_supported_on_device`**. There, a human fills the login with the Vault button in Pinework Desktop.

## `exec` — when the five verbs do not fit

`snapshot` reads the accessibility tree and keeps interactive roles. Real pages hide things from it: a `div` with a click handler, a closed shadow root, a cross-origin frame, a cookie banner. When `snapshot` cannot see the control, write the script yourself.

`--js` runs code in the page, `--js-file` runs it from a file, and `--cdp` sends one DevTools command. The timeout defaults to 30,000 ms, with a ceiling of 120,000 ms. The command prints what the script returned, plus anything the page logged. A script that throws exits non-zero, so `&&` chains stop.

Three that cover most of what you will need:

```bash
# 1. Read every link on the page.
pinework browser exec --js "[...document.querySelectorAll('a')].map(a => a.textContent.trim()).filter(Boolean)"

# 2. Click the control whose text matches — no ref, no snapshot.
pinework browser exec --js "const el = [...document.querySelectorAll('a,button')].find(e => /sign in/i.test(e.textContent)); if (!el) throw new Error('no match'); el.click(); el.textContent.trim()"

# 3. Wait for a selector, then screenshot it to a file.
pinework browser exec --timeout-ms 30000 --js "const deadline = Date.now() + 20000; let field = null; while (Date.now() < deadline && !field) { field = document.querySelector('input[type=email]'); if (!field) await new Promise(r => setTimeout(r, 200)); } if (!field) throw new Error('selector never appeared'); field.placeholder"
pinework browser exec --cdp Page.captureScreenshot --params '{"format":"png"}' --out "${TMPDIR:-/tmp}/page.png"
pinework file upload "${TMPDIR:-/tmp}/page.png" --task PIN-…
```

Writing the script:

- **The last expression is the answer.** `return` at the top level is a syntax error — end on the value instead, in parentheses if it is an object literal: `({ url: location.href })`.
- **`await` works at the top level.** That is how you wait for a selector, a fetch or an animation.
- **Keep the result small.** Map to the three fields you need; do not return a DOM node or the whole document.
- **`--js-file` for anything long.** Shell quoting eats scripts.

`--cdp` sends ONE Chrome DevTools command to the tab. Allowed: `Page.captureScreenshot`, `Page.navigate`, `Runtime.evaluate`, `Emulation.setDeviceMetricsOverride`, and all of `Input.*`, `DOM.*`, `Accessibility.*`. So a key press is `--cdp Input.dispatchKeyEvent --params '{"type":"keyDown","key":"Enter","code":"Enter","windowsVirtualKeyCode":13}'`. Anything else is refused and the error names the method — `Network.*`, `Storage.*` and `Fetch.*` would read the tab's cookies, `Target.*` and `Browser.*` would leave the tab.

`--out` writes a base64 reply, which is what a screenshot is, to a file instead of printing it.

**What is logged.** The panel shows the first 80 characters of your script and never its result, the same way `type` shows the field and never the text. Say what you did in a comment; the log alone will not tell the human what came back.

**What a script can read.** Anything on the page, including the value of a password box a human typed into that tab. Do not read fields you were not sent for, and never paste one into a comment.
