---
title: "vscode-statusbar"
weight: 40
---

# vscode-statusbar {#vscode-statusbar}

A VS Code status bar item that always shows the Claude usage `aiquota` measures: the same numbers as the Claude Code extension's *Account & usage* view, visible permanently instead of scrolling away.

It reads `Claude 30% · 5%` (session · weekly). Hover for every window, its reset time, the plan and the measurement time; click for the detail in the "aiquota" output channel. The item takes the status bar's warning colour from 75 % and its error colour from 90 % or when a window is exhausted. When the usage cannot be read it shows `Claude ?` with the reason in the tooltip, never a stale number.

## Design {#design}

VS Code extensions must be JavaScript, so the extension is a thin shell: it runs `aiquota status -o statusbar` and shows the one JSON object it prints (`text`, `tooltip`, `level`). Reading the credential, verdicts, thresholds and wording stay in the Go binary (`aiquota` v0.3.0 or newer), tested there; no secret passes through the extension. One process at a time, killed on timeout; a 60 s interval (minimum 30 s) and a cooldown that also covers clicks keep the usage endpoint from being hammered.

## Install {#install}

Download `aiquota-statusbar-<version>.vsix` from the [releases](https://github.com/go-aiquota/vscode-statusbar/releases) and run `code --install-extension aiquota-statusbar-<version>.vsix`. It is not published on the Marketplace. Settings: `aiquota.binaryPath` (default `~/.local/bin/aiquota`), `aiquota.intervalSeconds`, `aiquota.cooldownSeconds`, `aiquota.timeoutSeconds`, `aiquota.warnPercent`, `aiquota.criticalPercent`.
