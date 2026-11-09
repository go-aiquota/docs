---
title: "vscode-statusbar"
weight: 40
---

# vscode-statusbar {#vscode-statusbar}

A VS Code status bar item that always shows the Claude usage `aiquota` measures: the same numbers as the Claude Code extension's *Account & usage* view, visible permanently instead of scrolling away.

It shows one item per window, `Claude 5h 30%` and `7d 5%`, each coloured by its own band so the colour tells which window is hot; a per-model weekly window appears as a third item once it is above zero. Hover for every window, its reset time, the plan and the measurement time; click for the detail in the "aiquota" output channel.

The scale has 8 equal steps of 12.5 % (`aiquota.bands` for fewer): light green, dark green, light yellow, dark yellow, light orange, dark orange, then the theme's warning background (75 to 87.5 %) and error background (from 87.5 %). VS Code lets an extension use only those two backgrounds (`StatusBarItem.backgroundColor`), so every other step is a text colour with WCAG contrast of at least 3:1 against the default status bar (#007ACC in Dark+ and Light+, #68217A with no folder), asserted by a test in `tray`. On that mid-blue bar a text colour cannot be dark, so a "dark" step is the more saturated one. When the usage cannot be read it shows a grey `Claude ?` with the reason in the tooltip, never a stale number.

## Design {#design}

VS Code extensions must be JavaScript, so the extension is a thin shell: it runs `aiquota status -o statusbar` and shows the one JSON object it prints (`items` with `text`, `band`, `color`, `background`, and a `tooltip`). Reading the credential, verdicts, thresholds, bands, colours and wording stay in the Go binary (`aiquota` v0.4.0 or newer), tested there; no secret passes through the extension. One process at a time, killed on timeout; a 60 s interval (minimum 30 s) and a cooldown that also covers clicks keep the usage endpoint from being hammered.

## Install {#install}

Download `aiquota-statusbar-<version>.vsix` from the [releases](https://github.com/go-aiquota/vscode-statusbar/releases) and run `code --install-extension aiquota-statusbar-<version>.vsix`. It is not published on the Marketplace. Settings: `aiquota.binaryPath` (default `~/.local/bin/aiquota`), `aiquota.intervalSeconds`, `aiquota.cooldownSeconds`, `aiquota.timeoutSeconds`, `aiquota.bands`, `aiquota.warnPercent`, `aiquota.criticalPercent`.
