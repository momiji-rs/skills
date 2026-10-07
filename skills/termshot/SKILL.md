---
name: termshot
description: Render terminal output into a PNG, plain text, or JSON of the final screen with the termshot CLI (momiji-rs/termshot). Use it to screenshot a TUI or CLI command, check what a terminal app actually displays (layout, colours, cursor) without opening a terminal, make deterministic screenshots for READMEs, PRs, and CI golden tests, or render a PTY log or asciinema .cast someone sent.
argument-hint: "[log-or-cast | command]"
allowed-tools: Bash(termshot *) Bash(npx -y @momiji-rs/termshot *) Bash(uname *) Bash(tmux capture-pane *) Bash(tmux display *)
license: MIT
compatibility: macOS or Linux. Needs the termshot CLI from npm (@momiji-rs/termshot) or GitHub releases, not Homebrew's termshot; tmux is optional, for running TUIs.
metadata:
  termshot-version: "0.3.2"
---

# termshot

termshot replays the bytes a terminal program wrote (a raw PTY log or an asciinema
v2/v3 `.cast`) through an xterm-compatible screen model. It then writes the **final
screen** as a PNG, as text, or as JSON. It is headless: you don't need a terminal,
a window, or a display. The same input gives the same pixels on macOS and Linux.

It draws one frame and doesn't animate. It doesn't run the program for you, so
capture the output first (see below) and then render it.

On this machine: !`termshot --version 2>/dev/null || echo "termshot not installed"`, on !`uname -sm`.
Ours prints `termshot X.Y.Z`; if it prints anything else or isn't installed, see
[Install](#install) at the end. Arguments, if any: `$ARGUMENTS` (a log or cast to
render, or a command to screenshot).

## Choose a capture method

| Situation | Capture with | Render with |
|---|---|---|
| A command that runs and exits | `script` (a real PTY) | `termshot out.pty out.png` |
| A TUI that is still running, or one you drive with keys | tmux | `--lf-newline --size --cursor` |
| An existing asciinema recording | none | `termshot rec.cast out.png` |
| Plain text, `cmd > out.log`, or a text file | none | `--lf-newline` |

### A command that exits: `script`

The program has to see a TTY, or it drops colours and TUI mode. `script`'s
syntax differs between platforms:

```bash
# macOS
script -q session.pty sh -c 'your-command --flags'
# Linux (util-linux)
script -q -c 'your-command --flags' session.pty

termshot --size 100x30 session.pty session.png
```

Set the grid to the size the program drew for. `script` uses the current
terminal's size, so pin it if you need a particular size (for example,
`stty cols 100 rows 30` inside the `sh -c`).

### A running TUI: tmux

tmux ends each row with a bare LF, so pass `--lf-newline`. It also doesn't
record the cursor, so ask tmux for the cursor position:

```bash
tmux new-session -d -s shot -x 100 -y 30 'your-tui'
sleep 1                                   # let it draw; send keys with tmux send-keys
cursor=$(tmux display -p -t shot '#{?cursor_flag,#{cursor_x}#,#{cursor_y},none}')
tmux capture-pane -t shot -e -p \
  | termshot --lf-newline --size 100x30 --cursor "$cursor" - shot.png
tmux kill-session -t shot
```

`capture-pane` drops kitty graphics. To capture inline images, record through a
PTY with `script`.

## Check the screen instead of looking at it

To check what is on the screen, read text, not pixels. `--text` lays rows out
as `tmux capture-pane -p` does: one line per row, trailing spaces trimmed. Without
a PNG it skips most of the font work and runs about ten times faster:

```bash
termshot --text - session.pty | grep -q 'Saved'
termshot --text screen.txt --json screen.json session.pty screen.png   # all three
```

`--json` adds colours, attributes, and the cursor. Each row is a list of runs of
cells that look alike:

```json
{"cols":100,"rows":30,"cursor":{"col":2,"row":5,"shape":"block"},"lines":[
[{"col":0,"text":"ok","fg":"#00cd00","bg":"#111823","bold":true}, ...],
```

`cursor` is `null` when the program hides it. Attribute keys (`bold`, `italic`,
`underline`, `double_underline`, `strike`) appear only when they are set. Colours
are given as drawn, after reverse video and dim are applied.

When you need to see the layout, render the PNG and open it with your image-reading
tool. A smaller `--px` (such as 20) keeps the file small.

## Options you will use

| Option | Use |
|---|---|
| `-s, --size CxR` | Grid size. Default: the cast's size, else 100x30. Max 500x200 |
| `-p, --px N` | Font pixel height, 1–255, default 48. The image is about `cols × 0.46 × px` wide and `rows × px` tall (px 48 at 100x30 gives 2200x1440) |
| `--lf-newline` | Treat a bare LF as CR LF (tmux captures, `> file` logs, text files) |
| `--cursor COL,ROW` / `none` | Override the cursor position, counting from 0 |
| `--fallback-font FILE` | A font for characters the main font lacks, such as CJK (Noto Sans CJK `.ttc#N`) |
| `--fg` / `--bg #RRGGBB`, `--palette FILE` | Theme colours. The palette uses kitty's keys |
| `--padding N` | Margin in pixels around the grid |
| `-` | Read stdin as `<log>`, or write stdout as one output |

## Results to watch for

- **Exit codes:** 0 means success. 1 means a file couldn't be read or written, the
  cast is malformed, or the font is unusable. 2 means bad arguments. A failed run
  deletes the outputs it created.
- **A stderr hint about `--lf-newline`:** the log has LFs but no CRs, so it
  wasn't captured through a PTY. Rerun with `--lf-newline`, or the rows will
  stair-step.
- **Boxes where characters should be:** neither font has the glyph. For CJK, pass
  `--fallback-font`. For emoji, you need a monochrome outline font such as
  Noto Emoji, because colour emoji fonts are bitmaps.
- **A shifted or wrapped layout:** `--size` doesn't match the size the program
  drew for.

## Install

⚠️ **`brew install termshot` installs a different tool** (homeport/termshot).
Install from npm (Node 22 or newer, macOS or Linux):

```bash
npm install -g @momiji-rs/termshot        # puts termshot on PATH
npx -y @momiji-rs/termshot --version      # or run it without installing
```

Without Node, download the release binary instead:

```bash
v=0.3.2   # latest: gh release view -R momiji-rs/termshot --json tagName -q .tagName
case "$(uname -s)-$(uname -m)" in
  Darwin-*)       p=macos-universal ;;
  Linux-x86_64)   p=linux-x86_64-musl ;;
  Linux-aarch64)  p=linux-aarch64-musl ;;
esac
curl -fsSL "https://github.com/momiji-rs/termshot/releases/download/v$v/termshot-$v-$p.tar.gz" \
  | tar xz -C /tmp
mkdir -p ~/.local/bin && install -m 755 "/tmp/termshot-$v-$p/termshot" ~/.local/bin/termshot
```

It is one static binary with JetBrains Mono built in, so it needs no other files.
