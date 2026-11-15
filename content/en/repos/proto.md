---
title: "proto"
weight: 30
---

# proto {#proto}

The QuotaProvider gRPC contract, plus Secret/redact credential-safety types.

A <code>hashicorp/go-plugin</code> service (<code>Describe</code> + <code>FetchQuota</code>) with <code>AutoMTLS</code> so a credential never crosses the loopback socket in the clear, plus <code>secret.Secret</code> (a value that can't accidentally print via <code>%v</code>, a wrapped error, or JSON) and shape-based <code>redact</code> text scrubbing as a second line of defense over raw text that isn't already wrapped.

## Highlights {#highlights}

- `quotapb.QuotaProviderServer`: `Describe` (identity + login URL) and `FetchQuota`.
- A credential is `map<string,string>` (a whole cookie jar), not one opaque string.
- `secret.Secret` — the only way to read it back is `Secret.Reveal(func(string))`.
- `redact` — shape-based scrubbing for text that isn't already `Secret`-wrapped.
- `AutoMTLS` on every plugin connection — nothing crosses the loopback socket in the clear.

## Adding a provider {#adding-a-provider}

Implement the QuotaProvider contract and serve it as a go-plugin subprocess:

```go
func (p *myProvider) Describe(...) (*quotapb.ProviderInfo, error) { ... }
func (p *myProvider) FetchQuota(...) (*quotapb.QuotaSnapshot, error) { ... }

hcplugin.Serve(&hcplugin.ServeConfig{
    HandshakeConfig: plugin.Handshake,
    Plugins:         plugin.Map(p),
})
```

The reference implementation is [plugin-claude]({{< relref "plugin-claude.md" >}}). The host starts it as a separate subprocess, so a new provider is a new plugin, not a fork of the host.

## Install {#install}

```sh
go get github.com/go-aiquota/proto
```

Requires Go 1.26.4 or newer.

## Links {#links}

- Source — <https://github.com/go-aiquota/proto>
- API reference — <https://pkg.go.dev/github.com/go-aiquota/proto>
