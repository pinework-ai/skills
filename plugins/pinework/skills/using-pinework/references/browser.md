# The Desktop browser tab

A device run can look at a page and click through a UI. Six verbs drive the browser tab inside Pinework Desktop on this device (PIN-3019). `browser open <url>` opens or reuses the tab. `navigate <url>` moves the open tab. `snapshot` prints the page as refs you can act on. `click <@eN>` presses the element a ref names. `type <@eN> "text"` types into it. `exec` runs your own script in the page. `pinework browser <cmd> --help` has the flags.

Rules:

- **Refs come from `snapshot`.** Never guess `@e4`. Re-snapshot after the page changes, or the verb fails `stale_ref` and tells you to snapshot again.
- **Any verb starts Desktop hidden if it is not running**, then retries once (`sendDesktopVerb`, `apps/cli/src/runtime/desktop-control-client.ts`). No window opens and nothing steals focus. `app_unavailable` means one of three things: this machine is not a Mac, the app is not installed, or the installed build is older than the browser verbs. The error says which, and names the installed version (`appUnavailableHint`, `apps/cli/src/commands/browser.ts`). Do not retry: read the advice, then say what you found.
- **Testing a dev build? Say so.** Export `PINEWORK_DESKTOP_CHANNEL=dev` and the verbs talk to a checkout running `pnpm --filter desktop dev` instead of the installed app (PIN-3491). The two run side by side on their own sockets, in their own homes: the variable also moves the CLI to `~/.pinework-dev`, with its own login (PIN-3679). Without the variable you get the installed app, which is the right default. A dev channel is never auto-started — if nothing is serving it, the error says so and names the command to run.
- **The tab is the human's.** It is the same tab the right panel shows, not a private browser. So say what you did: post a comment naming the url and what you clicked, and paste the snapshot when it is the evidence.

This is not `agent-browser`. That drives a separate Chromium nobody can watch. These verbs drive the tab a human can open and see.

**Device runs only, and the device has to be a Mac.** Pinework Desktop is packaged for macOS and nothing else. A cloud run cannot reach these verbs at all: the sandbox has no Desktop to drive. On a device you install or update Desktop when a verb answers `app_unavailable`. In the cloud there is nothing to install, so use the browser tool your run already has, or move the work to a device run.

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
pinework browser exec --cdp Page.captureScreenshot --params '{"format":"png"}' --out /tmp/page.png
pinework file upload /tmp/page.png --task PIN-…
```

Writing the script:

- **The last expression is the answer.** `return` at the top level is a syntax error — end on the value instead, in parentheses if it is an object literal: `({ url: location.href })`.
- **`await` works at the top level.** That is how you wait for a selector, a fetch or an animation.
- **Keep the result small.** Map to the three fields you need; do not return a DOM node or the whole document.
- **`--js-file` for anything long.** Shell quoting eats scripts.

`--cdp` sends ONE Chrome DevTools command to the tab. Allowed: `Page.captureScreenshot`, `Page.navigate`, `Runtime.evaluate`, `Emulation.setDeviceMetricsOverride`, and all of `Input.*`, `DOM.*`, `Accessibility.*`. So a key press is `--cdp Input.dispatchKeyEvent --params '{"type":"keyDown","key":"Enter","code":"Enter","windowsVirtualKeyCode":13}'`. Anything else is refused and the error names the method — `Network.*`, `Storage.*` and `Fetch.*` would read the tab's cookies, `Target.*` and `Browser.*` would leave the tab.

`--out` writes a base64 reply, which is what a screenshot is, to a file instead of printing it.

**What is logged.** The panel shows the first 80 characters of your script and never its result, the same way `type` shows the field and never the text. Say what you did in a comment; the log alone will not tell the human what came back.

**What a script can read.** Anything on the page, including the value of a password box a human typed into that tab. One user per workspace today, so this is a stated limit rather than a leak — but do not read fields you were not sent for, and never paste one into a comment.
