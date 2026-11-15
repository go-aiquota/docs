---
title: "Install"
weight: 15
---

# Install {#install}

Written for someone who uses VS Code and nothing else: no Go, Node or git. Current versions: `aiquota` v0.9.0, the VS Code extension v0.3.2, `go-aiquota-plugin-claude` v0.2.1 (it ships in the `aiquota` archive). A status bar item at 100 % shows its reset time (`7d 100% → 2026-11-13 03:00`, local 24 h, with the date; aiquota v0.9.0 or newer).

## Before you start {#prerequisites}

Sign in to the Claude Code extension (or the CLI) in VS Code, as the same operating-system user that will run `aiquota`. `aiquota` reads that login (macOS Keychain item `Claude Code-credentials`, or `~/.claude/.credentials.json` / `$CLAUDE_CONFIG_DIR`) and sends it only to `api.anthropic.com`. Nothing passes through the extension. With Remote - SSH, WSL or Dev Containers, do every step on the remote machine.

## 1. Install aiquota {#aiquota}

Download the archive for your system from the [aiquota releases](https://github.com/go-aiquota/tray/releases/latest): `aiquota_<tag>_<os>_<arch>.tar.gz` (`.zip` on Windows), for Linux, macOS, Windows, FreeBSD, OpenBSD and NetBSD, amd64 and arm64, plus `SHA256SUMS` and a GitHub provenance attestation. Check the archive against `SHA256SUMS` (`shasum -a 256 -c --ignore-missing SHA256SUMS`, or `Get-FileHash` on Windows).

Put `aiquota`, `quotaclaim` and `go-aiquota-plugin-claude` (the provider plugin `aiquota` starts, looked up under that exact name) in one folder. `~/.local/bin` is searched first, then `$GOBIN`, `$GOPATH/bin`, `~/go/bin` (Windows: `aiquota.exe`, also `%LOCALAPPDATA%\aiquota`). For any other folder, set `aiquota.binaryPath` to the full path of `aiquota`.

On macOS a browser download is quarantined: run `xattr -d com.apple.quarantine` on each file.

With Go instead (the plugin is a separate module, and `go install` names it `plugin-claude`):

```sh
GOBIN=~/.local/bin go install github.com/go-aiquota/tray/cmd/aiquota@latest
GOBIN=~/.local/bin go install github.com/go-aiquota/tray/cmd/quotaclaim@latest
GOBIN=~/.local/bin go install github.com/go-aiquota/plugin-claude@latest
mv ~/.local/bin/plugin-claude ~/.local/bin/go-aiquota-plugin-claude
```

Linux, macOS and Windows binaries are run by CI on a runner of their own before a release is published; the BSD ones are built on every change but never run.

## 2. Install the extension {#extension}

It is not on the Marketplace or Open VSX. Download `aiquota-statusbar-<version>.vsix` from the [extension releases](https://github.com/go-aiquota/vscode-statusbar/releases/latest), then in the Extensions view open the `...` menu, choose **Install from VSIX...** and pick the file. Or, from a terminal: `code --install-extension aiquota-statusbar-<version>.vsix`.

## 3. Check {#check}

Run `aiquota status` in a terminal: it prints your account and one line per window. The status bar then shows one item per window, such as `Claude 5h 57%` and `7d 10%`. A grey `Claude ?` carries the reason in its tooltip (`aiquota not found at <path>`, `no go-aiquota-plugin-claude on PATH`, `token expired, open Claude Code`). The items are described in the [status bar reference]({{< relref "repos/vscode-statusbar.md" >}}).

## Update and uninstall {#update}

Update by installing the newer `.vsix` the same way and replacing the programs, then run "Developer: Reload Window". Uninstall from the Extensions view and delete the programs.

## Optional: the quotaclaim hook {#quotaclaim}

`quotaclaim` is a Claude Code hook that refuses a reply stating a usage limit without a fresh measurement. Add it to `~/.claude/settings.json`; the behaviour is in [the tray page]({{< relref "repos/tray.md" >}}#quotaclaim).

```json
{"hooks": {
  "Stop":         [{"hooks": [{"type": "command", "command": "~/.local/bin/quotaclaim"}]}],
  "SubagentStop": [{"hooks": [{"type": "command", "command": "~/.local/bin/quotaclaim"}]}]
}}
```
