---
description: Command-line interface for XCHandles
---

# CLI

The [slot-machine](https://github.com/Yakuhito/slot-machine) repository provides a CLI for interacting with XCHandles on-chain. Puzzle source lives in slot-machine; spend construction uses [chia-wallet-sdk](https://github.com/Yakuhito/chia-wallet-sdk) drivers.

### Invocation

```bash
cargo r xchandles <subcommand> [flags]
```

Or, after building: `slot-machine xchandles <subcommand> [flags]`.

### Prerequisites

* **Sage wallet** - the Sage RPC should be running locally.
* **Coinset RPC** - used for chain sync and broadcasting (mainnet or testnet11).
* **Neighbor API** - register, extend, and expire commands need left/right slot neighbors. By default the CLI queries `https://api.xchandles.com`; use `--local` to query a local indexer started with `xchandles listen`.

### Common Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--testnet11` | `false` | Use testnet11 consensus constants and coinset |
| `--fee` | `0.0025` | Transaction fee in XCH |
| `--local` | `false` | Use local SQLite index (`data.db`) instead of remote API |
| `--log` | `false` | Write spend bundle debug output to `sb.debug` |
| `--debug` | `false` | Debug signing mode (prompt for private key) |

### License

slot-machine is licensed under the [Prosperity Public License](https://prosperitylicense.com/). Commercial use requires a license after the trial period.

### Further Reading

* [XCHandles Commands](xchandles.md) — full command reference
