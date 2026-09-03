# tray

The menu-bar host app — per-account tray items, plugin manager, isolated onboarding.

Renders one tray item per account (a Stats-style <code>"46%/10%"</code> text display), launches and polls each account's provider plugin over the <code>go-aiquota/proto</code> gRPC contract, and drives a per-account isolated embedded-browser window (a fresh, non-persistent data store each time) for onboarding — so no cookie is ever copied out of a real browser by hand.

## Highlights

- One tray item **per account**, text-only (no icon clutter) — <code>session%/weekly%</code>.
- Plugin manager: launches and polls provider subprocesses over gRPC.
- Per-account **isolated** onboarding (WKWebView non-persistent data store on macOS).
- A credential never touches <code>accounts.json</code> — metadata only, OS keyring for the rest.
- Testable end-to-end without a mouse: the tray's own state machine is driven by the exact call (<code>MenuItem.Activate()</code>) a real click makes.

## Install

```sh
go get github.com/go-aiquota/tray
```

Requires Go 1.26.4 or newer.

## Links

- Source — <https://github.com/go-aiquota/tray>
- API reference — <https://pkg.go.dev/github.com/go-aiquota/tray>
