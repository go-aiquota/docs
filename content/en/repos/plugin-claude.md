---
title: "plugin-claude"
weight: 40
---

# plugin-claude {#plugin-claude}

The reference QuotaProvider plugin — Claude Max / Team Premium / Team Standard.

Calls the same undocumented endpoint claude.ai's own web app calls to show the usage bar under Settings → Usage, authenticated purely by the account's session cookies. Because claude.ai's login sits behind Cloudflare's interactive JS challenge, onboarding always drives a real embedded browser — the plugin itself only ever sees the resulting cookies and org UUID.

## Claude Code source {#claude-code}

With <code>Credential = {"source": "claude-code"}</code> the plugin reads **Claude Code's own login** at fetch time (macOS Keychain item <code>Claude Code-credentials</code> via <code>/usr/bin/security</code>, else <code>~/.claude/.credentials.json</code>) and calls <code>GET https://api.anthropic.com/api/oauth/usage</code> with the bearer token and <code>anthropic-beta: oauth-2025-04-20</code> — the same request Claude Code makes — returning the windows with the labels and order of the VS Code *Account & usage* panel (<code>Session (5hr)</code>, <code>Weekly (7 day)</code>, <code>Weekly Sonnet</code>, <code>Weekly &lt;model&gt;</code>). It never refreshes or rotates the token (an expired one is reported as "token expired, open Claude Code"), holds it in a <code>secret.Secret</code>, redacts it from every error, talks to that one host over TLS 1.2+ with no redirects and a 1 MiB response cap, and has fuzz targets for both decoders.

## Highlights {#highlights}

- `GET /api/organizations/{org_uuid}/usage`, cookie-authenticated, found by watching a real session's own network traffic.
- Needs `org_uuid` alongside the cookie jar — captured once at onboarding time, since no scripted request can rediscover it from cookies alone.
- Talks to claude.ai over a TLS-fingerprinted ([go-browserhttp](https://github.com/go-browserhttp/browserhttp)) client for the one authenticated JSON call — login itself needs a real browser, this doesn't.
- A credential missing `org_uuid` fails clearly rather than guessing.

## Install {#install}

```sh
go get github.com/go-aiquota/plugin-claude
```

Requires Go 1.26.4 or newer.

## Links {#links}

- Source — <https://github.com/go-aiquota/plugin-claude>
- API reference — <https://pkg.go.dev/github.com/go-aiquota/plugin-claude>
