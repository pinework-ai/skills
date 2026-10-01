# The Desktop terminal tab

A device run can read the shell a human has open in Pinework Desktop, and type into it (PIN-3055). `terminal list` shows the open shells. `terminal read` prints the tail of the scrollback as plain text. `terminal send "text"` types it and presses Return. `terminal run "command"` always presses Return. `pinework terminal <cmd> --help` has the flags.

Rules:

- **Run your own commands with Bash, as before.** Use the terminal for the two things Bash cannot do: answer a prompt a running process is waiting on (a `y/N`, a TUI), and read what a human's long-running command printed.
- **Never answer a `sudo`, `ssh` or database password prompt.** That gate is not approved yet. Ask the human in a comment instead.
- **It is the human's shell.** Every `send` lands in front of them, under a grey line `── @you is typing ──`. So say what you did in a comment.
- **Read, send, read.** `send` answers with a character count, never the output. Run `terminal read` after it to see what the shell did.
- **`--session` is optional.** With one shell open, that is the one. With none, a human has to open the terminal tab first; you cannot start a shell. With several, the error names them.
- **The log never holds what you typed.** It records `send` and the session. Your text still reaches the screen if the program echoes it, as any keyboard's would.
- **`truncated` means the start is missing.** The buffer keeps 512 KiB, about 52 bytes a line over 10,000 lines. A large `--lines` is rarely whole, and the read says so.
- **A human keystroke takes the shell back at once.** Your next `send` prints the grey line again.

Device runs only, on a Mac. Starting Desktop, `app_unavailable` and `PINEWORK_DESKTOP_CHANNEL=dev` work exactly as in `references/browser.md`.
