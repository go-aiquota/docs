# plugin-claude

The reference QuotaProvider plugin — Claude Max / Team Premium / Team Standard.

Calls the same undocumented endpoint claude.ai's own web app calls to show the usage bar under Settings → Usage, authenticated purely by the account's session cookies. Because claude.ai's login sits behind Cloudflare's interactive JS challenge, onboarding always drives a real embedded browser — the plugin itself only ever sees the resulting cookies and org UUID.

## Highlights

- `GET /api/organizations/{org_uuid}/usage`, cookie-authenticated, found by watching a real session's own network traffic.
- Needs `org_uuid` alongside the cookie jar — captured once at onboarding time, since no scripted request can rediscover it from cookies alone.
- Talks to claude.ai over a TLS-fingerprinted ([go-browserhttp](https://github.com/go-browserhttp/browserhttp)) client for the one authenticated JSON call — login itself needs a real browser, this doesn't.
- A credential missing `org_uuid` fails clearly rather than guessing.

## Install

```sh
go get github.com/go-aiquota/plugin-claude
```

Requires Go 1.26.4 or newer.

## Links

- Source — <https://github.com/go-aiquota/plugin-claude>
- API reference — <https://pkg.go.dev/github.com/go-aiquota/plugin-claude>
