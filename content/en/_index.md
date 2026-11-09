---
title: "Home"
weight: 10
---

# go-aiquota {#go-aiquota}

A cross-platform menu-bar app that monitors AI usage quotas across multiple accounts in parallel.

`go-aiquota` is a cross-platform menu-bar app that watches AI usage quotas — session and weekly limits — across multiple accounts in parallel, through a small gRPC plugin contract so a new provider is a new plugin, not a fork. Onboarding drives a real, isolated embedded browser to the provider's own login page; a credential is never logged and lives only in the OS keyring.

From the terminal, `aiquota status` measures every limit live (exit 0 available, 1 exhausted, 3 unknown), and the `quotaclaim` Claude Code hook refuses a reply that states a usage limit without a fresh measurement — see [tray]({{< relref "repos/tray.md" >}}#cli).

## Repositories {#repositories}

<div class="repo-grid" markdown>
<a class="repo-card" href='{{< relref "repos/tray.md" >}}'><code>tray</code><br><small>The menu-bar host app — per-account tray items, plugin manager, isolated onboarding.</small></a>
<a class="repo-card" href='{{< relref "repos/proto.md" >}}'><code>proto</code><br><small>The QuotaProvider gRPC contract, plus Secret/redact credential-safety types.</small></a>
<a class="repo-card" href='{{< relref "repos/plugin-claude.md" >}}'><code>plugin-claude</code><br><small>The reference QuotaProvider plugin — Claude Max / Team Premium / Team Standard.</small></a>
</div>
