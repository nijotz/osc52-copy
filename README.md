# osc52-copy

Copy text to the local clipboard from anywhere using the [OSC 52](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html#h3-Operating-System-Commands) terminal escape sequence. Works over SSH without X11/Wayland forwarding.

## Usage

```bash
echo hello | osc52-copy
osc52-copy "some text"
```

## Installation

Drop the script somewhere on your `$PATH` and make it executable:

```bash
install -m 755 osc52-copy ~/.local/bin/
```

## Requirements

- A terminal emulator that supports OSC 52 (iTerm2, kitty, WezTerm, recent xterm, Alacritty with `terminal.osc52` enabled, etc.).
- Standard tools: `bash`, `base64`, `tr`, `printf`.

## Inside tmux

The script auto-detects `$TMUX` and wraps the sequence in tmux's DCS passthrough so it reaches the outer terminal. tmux 3.3+ requires this in your config:

```tmux
set -g allow-passthrough on
```

Without that, the sequence is silently dropped.

## How it works

OSC 52 lets a program tell the terminal "put this base64-encoded text in the clipboard." The script:

1. Reads input from stdin or arguments.
2. Base64-encodes it (newlines stripped, since OSC sequences cannot contain them).
3. Wraps it as `ESC ] 52 ; c ; <base64> BEL` (`c` selects the system clipboard).
4. Inside tmux, additionally wraps in `ESC P tmux; ... ESC \` with every embedded ESC doubled, so tmux's passthrough forwards it intact.
5. Writes the sequence to `/dev/tty` so it works even when stdout is piped or redirected.

## Why not [theimpostor/osc](https://github.com/theimpostor/osc)?

`osc` is a more featureful Go tool (paste, multiple clipboard targets, tty override). `osc52-copy` is a few lines of bash that fits into dotfiles without an extra binary install.
