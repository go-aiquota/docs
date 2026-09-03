# proto

The QuotaProvider gRPC contract, plus Secret/redact credential-safety types.

A <code>hashicorp/go-plugin</code> service (<code>Describe</code> + <code>FetchQuota</code>) with <code>AutoMTLS</code> so a credential never crosses the loopback socket in the clear, plus <code>secret.Secret</code> (a value that can't accidentally print via <code>%v</code>, a wrapped error, or JSON) and shape-based <code>redact</code> text scrubbing as a second line of defense over raw text that isn't already wrapped.

## Highlights

- `quotapb.QuotaProviderServer`: `Describe` (identity + login URL) and `FetchQuota`.
- A credential is `map<string,string>` (a whole cookie jar), not one opaque string.
- `secret.Secret` — the only way to read it back is `Secret.Reveal(func(string))`.
- `redact` — shape-based scrubbing for text that isn't already `Secret`-wrapped.
- `AutoMTLS` on every plugin connection — nothing crosses the loopback socket in the clear.

## Install

```sh
go get github.com/go-aiquota/proto
```

Requires Go 1.26.4 or newer.

## Links

- Source — <https://github.com/go-aiquota/proto>
- API reference — <https://pkg.go.dev/github.com/go-aiquota/proto>
