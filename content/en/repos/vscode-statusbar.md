---
title: "vscode-statusbar"
weight: 40
---

# vscode-statusbar {#vscode-statusbar}

A VS Code status bar item that always shows the Claude usage `aiquota` measures: the same numbers as the Claude Code extension's *Account & usage* view, visible permanently instead of scrolling away.

It shows one item per window, `Claude 5h 30%` and `7d 5%`, each coloured by its own band so the colour tells which window is hot; a per-model weekly window appears as a third item once it is above zero. Hover for every window, its reset time, the plan and the measurement time; click for the detail in the "aiquota" output channel.

The scale has 8 equal steps of 12.5 % (`aiquota.bands` for fewer): light green, dark green, light yellow, dark yellow, light orange, dark orange, then the theme's warning background (75 to 87.5 %) and error background (from 87.5 %). VS Code lets an extension use only those two backgrounds (`StatusBarItem.backgroundColor`), so every other step is a text colour with WCAG contrast of at least 3:1 against the default status bar (#007ACC in Dark+ and Light+, #68217A with no folder), asserted by a test in `tray`. On that mid-blue bar a text colour cannot be dark, so a "dark" step is the more saturated one. When the usage cannot be read it shows a grey `Claude ?` with the reason in the tooltip, never a stale number.

## Design {#design}

VS Code extensions must be JavaScript, so the extension is a thin shell: it runs `aiquota status -o statusbar --render 1` and draws exactly the items that description lists (per item `id`, `text`, `tooltip`, `color`, `background`, `priority`, `alignment`, `command`, plus a `detail` text and `poll_seconds`). Reading the credential, verdicts, thresholds, bands, colours, labels and the item list stay in the Go binary (`aiquota` v0.6.0 or newer; the extension keeps asking for render 1, so a newer binary keeps working), tested there; whatever fails, the item reads `Claude ?` and never goes empty; no secret passes through the extension. One process at a time, killed on timeout; a 60 s interval (minimum 30 s) and a cooldown that also covers clicks keep the usage endpoint from being hammered.

## Changing the look: edit Go, go install, click {#tuning}

The extension is a frozen, generic renderer (v0.3.0): the number of items, their order, labels, colours, bands, click actions, thresholds and the poll interval all come from `aiquota`. To change any of them, edit `aiquota` in [tray](https://github.com/go-aiquota/tray), `go install` it, then click the item or run "aiquota: Refresh now" (or wait one poll). No window reload is needed, which matters because a reload restarts the extension host that every extension of the window, Claude Code included, shares; VS Code has no per-extension restart. The settings `warnPercent`, `criticalPercent`, `bands` and `intervalSeconds` are passed to the binary only when you set them; otherwise its defaults rule.

## Install {#install}

Download `aiquota-statusbar-<version>.vsix` from the [releases](https://github.com/go-aiquota/vscode-statusbar/releases) and run `code --install-extension aiquota-statusbar-<version>.vsix`. It is not published on the Marketplace. Settings: `aiquota.binaryPath` (default `~/.local/bin/aiquota`), `aiquota.intervalSeconds`, `aiquota.cooldownSeconds`, `aiquota.timeoutSeconds`, `aiquota.bands`, `aiquota.statusbar.alignment` and `aiquota.statusbar.priority` (the items take that base priority minus 0.001 steps so they stay contiguous), `aiquota.warnPercent`, `aiquota.criticalPercent`. After installing a new version, run "Developer: Reload Window".
