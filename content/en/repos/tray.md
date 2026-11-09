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

<code>aiquota status [--account A] [-o json|text]</code> loads the Claude plugin and asks **live** for every window (5 h session, weekly, any per-model window), printing used %, the reset time (local, 24 h) and the plan. Exit status: **0** available, **1** exhausted, **3** UNKNOWN (could not read — never reported as available or exhausted), 2 usage error. Every run writes a stamp (time, windows, verdicts, no credential) to <code>os.UserConfigDir()/aiquota/last-status.json</code>.

<code>quotaclaim</code> is a Claude Code <code>Stop</code> / <code>SubagentStop</code> hook that refuses a reply asserting a usage-limit state ("weekly limit", "hit your limit", "resets Nov 12", "limite hebdomadaire", "bloqué jusqu'au") unless a stamp younger than <code>--max-age</code> (default 10 minutes) backs it; it blocks with the reason naming <code>aiquota status</code>. An <code>unknown</code> stamp backs neither "available" nor "exhausted".

The matcher claims only statements about Claude's own usage ("5-hour limit", "weekly limit", "limite hebdomadaire", "hit your ... limit", "usage limit", "quota Claude"). Gateway vocabulary (soft/hard quota, <code>rate_limit_exceeded</code>, Retry-After, budgets, GitHub API rate limits) and denials are never claims. With no usable measurement (none, stale, UNKNOWN) a claim passes only if its own sentence says "unknown" or "not measured". <code>quotaclaim scan</code> replays the matcher over <code>~/.claude/projects</code> (read-only).

<code>aiquota status</code> keeps the plugin client's own log (handshake, mTLS, stdio) quiet at Warn; <code>--debug</code> shows it on stderr for diagnosis.

By default <code>aiquota status</code> shows **the same view as Claude Code**: it queries Claude Code's own login (plugin-claude's "Claude Code" source) and prints the rows of the VS Code extension's *Account & usage* panel with the panel's labels, order, whole percentages and reset wording (<code>Session (5hr)  37%  Resets at 14:00</code>, <code>Weekly (7 day)  52%  Resets Thu, Nov 12 at 03:00</code>, ...), so the two compare line by line. Stored claude.ai accounts are the fallback when no Claude Code login exists, or are selected with <code>--account</code>. The token is never refreshed (Claude Code renews its own login): once expired the status is UNKNOWN, "token expired, open Claude Code". The read goes through <code>/usr/bin/security</code>, which the Keychain item's ACL already trusts, so there is no dialog and nothing to re-approve after a reinstall.

```sh
GOBIN=~/.local/bin go install github.com/go-aiquota/tray/cmd/aiquota@latest
GOBIN=~/.local/bin go install github.com/go-aiquota/tray/cmd/quotaclaim@latest
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
