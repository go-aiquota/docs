---
title: "vscode-statusbar"
weight: 40
---

# vscode-statusbar {#vscode-statusbar}

A VS Code status bar item that always shows the Claude usage `aiquota` measures: the same numbers as the Claude Code extension's *Account & usage* view, visible permanently instead of scrolling away.

It shows one item per window, `Claude 5h 57%` and `7d 10%`, then each other window the provider returns (a per-model weekly window such as Fable, a 0 % one included), each coloured by its own band so the colour tells which window is hot. A window at 100 % shows its reset in the item itself (`7d 100% → 2026-11-13 03:00`, local 24 h, always with the date; aiquota v0.9.0 or newer). Hover for every window, its reset time, the plan and the measurement time; click for the detail in the "aiquota" output channel.

The scale has 8 equal steps of 12.5 % (`aiquota.bands` for fewer): light green, dark green, light yellow, dark yellow, light orange, dark orange, then the theme's warning background (75 to 87.5 %) and error background (from 87.5 %). VS Code lets an extension use only those two backgrounds (`StatusBarItem.backgroundColor`), so every other step is a text colour with WCAG contrast of at least 3:1 against the default status bar (#007ACC in Dark+ and Light+, #68217A with no folder), asserted by a test in `tray`. On that mid-blue bar a text colour cannot be dark, so a "dark" step is the more saturated one. When the usage cannot be read it shows a grey `Claude ?` with the reason in the tooltip, never a stale number.

## Design {#design}

VS Code extensions must be JavaScript, so the extension is a thin shell: it runs `aiquota status -o statusbar --render 1` and draws exactly the items that description lists (per item `id`, `text`, `tooltip`, `color`, `background`, `priority`, `alignment`, `command`, plus a `detail` text and `poll_seconds`). Reading the credential, verdicts, thresholds, bands, colours, labels and the item list stay in the Go binary (`aiquota` v0.6.0 or newer; the extension keeps asking for render 1, so a newer binary keeps working), tested there; whatever fails, the item reads `Claude ?` and never goes empty; no secret passes through the extension. One process at a time, killed on timeout; a 60 s interval (minimum 30 s) and a cooldown that also covers clicks keep the usage endpoint from being hammered.

## Changing the look: edit Go, go install, click {#tuning}

The extension is a frozen, generic renderer (v0.3.2 is the latest, installed in a real VS Code by CI): the number of items, their order, labels, colours, bands, click actions, thresholds and the poll interval all come from `aiquota`. To change any of them, edit `aiquota` in [tray](https://github.com/go-aiquota/tray), `go install` it, then click the item or run "aiquota: Refresh now" (or wait one poll). No window reload is needed, which matters because a reload restarts the extension host that every extension of the window, Claude Code included, shares; VS Code has no per-extension restart. The settings `warnPercent`, `criticalPercent`, `bands` and `intervalSeconds` are passed to the binary only when you set them; otherwise its defaults rule.

## Install {#install}

Written for someone who uses VS Code and nothing else (no Go, Node or git). The full guide, with every per-OS detail, is the [README's Install section](https://github.com/go-aiquota/vscode-statusbar#install); in short:

1. **Prerequisites.** The Claude Code extension (or CLI) signed in, as the same operating-system user that will run `aiquota`: it reads that login (macOS Keychain item `Claude Code-credentials`, or `~/.claude/.credentials.json` / `$CLAUDE_CONFIG_DIR`) and sends it only to `api.anthropic.com`. Nothing passes through the extension. With Remote - SSH, WSL or Dev Containers, do it on the remote machine.
2. **Install aiquota.** Download the archive for your system from the [aiquota releases](https://github.com/go-aiquota/tray/releases/latest) (`aiquota_<tag>_<os>_<arch>.tar.gz`, `.zip` on Windows; Linux, macOS, Windows, FreeBSD, OpenBSD and NetBSD, amd64 and arm64) and `SHA256SUMS`, and check the archive against it (`shasum -a 256 -c --ignore-missing SHA256SUMS`, or `Get-FileHash` on Windows). Put `aiquota` and `go-aiquota-plugin-claude` (the provider plugin `aiquota` starts) in one folder: `~/.local/bin` is searched first, then `$GOBIN`, `$GOPATH/bin`, `~/go/bin` (Windows: `aiquota.exe`, also `%LOCALAPPDATA%\aiquota`); for any other folder set `aiquota.binaryPath` to the full path of `aiquota`. On macOS a browser download is quarantined: `xattr -d com.apple.quarantine` on both files. With Go instead: `go install github.com/go-aiquota/tray/cmd/aiquota@latest` and `go install github.com/go-aiquota/plugin-claude@latest`, the latter renamed to `go-aiquota-plugin-claude`. Linux, macOS and Windows are tested by CI on a runner of their own; the BSD binaries are built on every change but never run.
3. **Install the extension.** It is not on the Marketplace or Open VSX. Download `aiquota-statusbar-<version>.vsix` from the [releases](https://github.com/go-aiquota/vscode-statusbar/releases/latest), then in the Extensions view open the `...` menu, **Install from VSIX...**, and pick it; or run `code --install-extension aiquota-statusbar-<version>.vsix`.
4. **Check.** `aiquota status` in a terminal prints your account and one line per window; the status bar shows one item per window. A grey `Claude ?` carries the reason in its tooltip (`aiquota not found at <path>`, `no go-aiquota-plugin-claude on PATH`, `token expired, open Claude Code`).

Update by installing the newer `.vsix` the same way and replacing the two programs; uninstall from the Extensions view and delete the programs.

Settings: `aiquota.binaryPath` (default: the first `aiquota` found in the folders above), `aiquota.intervalSeconds`, `aiquota.cooldownSeconds`, `aiquota.timeoutSeconds`, `aiquota.bands`, `aiquota.statusbar.alignment` and `aiquota.statusbar.priority` (the items take that base priority minus 0.001 steps so they stay contiguous), `aiquota.warnPercent`, `aiquota.criticalPercent`. After installing a new version, run "Developer: Reload Window".
