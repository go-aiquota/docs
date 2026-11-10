---
title: "tray"
weight: 20
---

# tray {#tray}

The menu-bar host app — per-account tray items, plugin manager, isolated onboarding.

Renders one tray item per account (a Stats-style <code>"46%/10%"</code> text display), launches and polls each account's provider plugin over the <code>go-aiquota/proto</code> gRPC contract, and drives a per-account isolated embedded-browser window (a fresh, non-persistent data store each time) for onboarding — so no cookie is ever copied out of a real browser by hand.

## Highlights {#highlights}

- One tray item **per account**, text-only (no icon clutter) — <code>session%/weekly%</code>.
- Plugin manager: launches and polls provider subprocesses over gRPC.
- Per-account **isolated** onboarding (WKWebView non-persistent data store on macOS).
- A credential never touches <code>accounts.json</code> — metadata only, OS keyring for the rest.
- Testable end-to-end without a mouse: the tray's own state machine is driven by the exact call (<code>MenuItem.Activate()</code>) a real click makes.

## Command line: aiquota and quotaclaim {#cli}

<code>aiquota status [--account A] [-o json|text|statusbar]</code> loads the Claude plugin and asks **live** for every window (5 h session, weekly, any per-model window), printing used %, the reset time (local, 24 h) and the plan. Exit status: **0** available, **1** exhausted, **3** UNKNOWN (could not read — never reported as available or exhausted), 2 usage error. Every run writes a stamp (time, windows, verdicts, no credential) to <code>os.UserConfigDir()/aiquota/last-status.json</code>.

**One reading for every process** (v0.7.1). The usage endpoint rate limits, and a machine runs several editor windows, the CLI and the <code>quotaclaim</code> gate, so they share one reading kept next to the stamp (<code>cache-*.json</code>, no credential): within <code>--min-interval</code> (default 1 minute) a process reuses the last good reading and shows its real measurement time; one process at a time fetches (a lock file); a <code>Retry-After</code> is written down and honoured by all of them; and while the endpoint cannot be asked the last good reading is served marked <code>stale</code> (its age shown) for up to <code>--max-stale</code> (default 10 minutes), after which the answer is UNKNOWN. <code>--fresh</code> fetches now, still respecting a <code>Retry-After</code>; <code>--min-interval 0</code> disables sharing. <code>quotaclaim</code> still checks its max-age against the real measurement time.

<code>-o statusbar</code> (v0.3.0, colours since v0.4.0) prints one JSON object <code>{version, text, tooltip, level, verdict}</code> for an editor status bar: <code>text</code> is <code>Claude 30% · 5%</code> (session · weekly) or <code>Claude ?</code> when UNKNOWN (the reasons are in the tooltip, never a number); <code>level</code> is <code>ok</code>, <code>warn</code>, <code>error</code> or <code>unknown</code> (<code>--warn 75</code>, <code>--critical 90</code> on the worst window; an exhausted window is always <code>error</code>). A client asks for the contract it understands with <code>--format-version</code>: 1 (the default, exactly the object above) or 2 (v0.5.0), which also carries <code>items</code>, one per window (session, weekly, and every other window the endpoint returns, a 0 % one included), each with <code>text</code>, <code>percent</code>, <code>band</code> (0 to 7, equal steps of 12.5 %; <code>--bands N</code>, 2 to 8, for coarser steps; -1 when unknown), <code>color</code> (hex) and <code>background</code> (<code>none</code>, <code>warning</code>, <code>error</code>). The 8 steps are light green, dark green, light yellow, dark yellow, light orange, dark orange, then the theme's warning and error backgrounds: VS Code's extension API honours only those two backgrounds, so every other step is a text colour with WCAG contrast of at least 3:1 against the default status bar (#007ACC in Dark+ and Light+, #68217A with no folder), asserted by a Go test. The [VS Code extension]({{< relref "vscode-statusbar.md" >}}) is its consumer.

<code>quotaclaim</code> is a Claude Code <code>Stop</code> / <code>SubagentStop</code> hook that refuses a reply asserting a usage-limit state ("weekly limit", "hit your limit", "resets Nov 12", "limite hebdomadaire", "bloqué jusqu'au") unless a stamp younger than <code>--max-age</code> (default 10 minutes) backs it; it blocks with the reason naming <code>aiquota status</code>. An <code>unknown</code> stamp backs neither "available" nor "exhausted".

The matcher claims only statements about Claude's own usage ("5-hour limit", "weekly limit", "limite hebdomadaire", "hit your ... limit", "usage limit", "quota Claude"). Gateway vocabulary (soft/hard quota, <code>rate_limit_exceeded</code>, Retry-After, budgets, GitHub API rate limits) and denials are never claims. With no usable measurement (none, stale, UNKNOWN) a claim passes only if its own sentence says "unknown" or "not measured". <code>quotaclaim scan</code> replays the matcher over <code>~/.claude/projects</code> (read-only).

<code>aiquota status</code> keeps the plugin client's own log (handshake, mTLS, stdio) quiet at Warn; <code>--debug</code> shows it on stderr for diagnosis.

By default <code>aiquota status</code> shows **every window the usage endpoint returns** (v0.7.0): it queries Claude Code's own login (plugin-claude's "Claude Code" source) and prints known windows with the panel's labels, order, whole percentages and reset wording (<code>Session (5hr)  37%  Resets at 14:00</code>, <code>Weekly (7 day)  52%  Resets Thu, Nov 12 at 03:00</code>, ...), Opus included, plus any other window with a label derived from its key and any <code>limits[]</code> entry the top level did not already report. A window listed without a value is <code>–</code> with verdict <code>n/a</code>: never a number, and outside the overall verdict. <code>--overlay</code> reproduces exactly the rows of the VS Code extension's *Account & usage* panel, so the two compare line by line, and <code>-o json --raw</code> prints the usage response as received (it holds no credential). Stored claude.ai accounts are the fallback when no Claude Code login exists, or are selected with <code>--account</code>. The token is never refreshed (Claude Code renews its own login): once expired the status is UNKNOWN, "token expired, open Claude Code". The read goes through <code>/usr/bin/security</code>, which the Keychain item's ACL already trusts, so there is no dialog and nothing to re-approve after a reinstall.

**Install without Go** (v0.8.0). Each [release](https://github.com/go-aiquota/tray/releases) carries one archive per platform, `aiquota_<tag>_<os>_<arch>.tar.gz` (`.zip` on Windows), for Linux, macOS, Windows, FreeBSD, OpenBSD and NetBSD, amd64 and arm64, plus `SHA256SUMS` and a GitHub provenance attestation. An archive holds `aiquota`, `quotaclaim` and `go-aiquota-plugin-claude` (the Claude provider `aiquota` starts, looked up on PATH under that exact name): put the three in one folder on your PATH after checking the archive against `SHA256SUMS`. CI builds all twelve on every pull request, but only Linux, macOS and Windows are tested on a runner; the three BSDs are built and never run. For the VS Code side, see the [vscode-statusbar install guide]({{< relref "vscode-statusbar.md" >}}).

**Install with Go**, the plugin being a separate module (`go install` names it `plugin-claude`: rename it to `go-aiquota-plugin-claude`):

```sh
GOBIN=~/.local/bin go install github.com/go-aiquota/tray/cmd/aiquota@latest
GOBIN=~/.local/bin go install github.com/go-aiquota/tray/cmd/quotaclaim@latest
GOBIN=~/.local/bin go install github.com/go-aiquota/plugin-claude@latest
mv ~/.local/bin/plugin-claude ~/.local/bin/go-aiquota-plugin-claude
```

Hook configuration (<code>~/.claude/settings.json</code>):

```json
{"hooks": {
  "Stop":         [{"hooks": [{"type": "command", "command": "~/.local/bin/quotaclaim"}]}],
  "SubagentStop": [{"hooks": [{"type": "command", "command": "~/.local/bin/quotaclaim"}]}]
}}
```

## Install {#install}

```sh
go get github.com/go-aiquota/tray
```

Requires Go 1.26.4 or newer.

## Links {#links}

- Source — <https://github.com/go-aiquota/tray>
- API reference — <https://pkg.go.dev/github.com/go-aiquota/tray>
