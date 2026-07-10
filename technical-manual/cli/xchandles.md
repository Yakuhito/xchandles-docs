---
description: XCHandles CLI command reference
---

# XCHandles Commands

All commands are invoked as `cargo run -- xchandles <command> [flags]`.

---

## Launch

### initiate-launch

Creates the XCHandles registry singleton, price singleton (medieval vault), state scheduler, and premine payment CAT.

```bash
cargo run -- xchandles initiate-launch \
  --pubkeys <comma-separated-hex-pubkeys> \
  -m <threshold> \
  --payout-address <xch/txch address> \
  [--relative-block-height 32] \
  [--registration-period 31557600] \
  [--testnet11] \
  [--fee 0.0025]
```

| Flag | Default | Description |
|------|---------|-------------|
| `--pubkeys` | (required) | Comma-separated price singleton signer pubkeys |
| `-m` | (required) | Multisig threshold |
| `--payout-address` | (required) | Where precommit refunds are sent |
| `--relative-block-height` | `32` | Blocks between precommit and register |
| `--registration-period` | `31557600` | Registration period in seconds (~1 year) |

Outputs: registry `launcher_id`, price singleton id, premine payment asset id.

### continue-launch

Registers premine handles from CSV and mints name NFTs.

```bash
cargo run -- xchandles continue-launch \
  --launcher-id <hex> \
  --payment-asset-id <hex> \
  --royalty-address <address> \
  --handles-per-spend <n> \
  [--start-time <unix>] \
  [--skip 0] \
  [--royalty-basis-points 1000] \
  [--registration-period 31557600] \
  [--testnet11] \
  [--fee 0.0025]
```

Reads `xchandles_premine_testnet11.csv` (or mainnet equivalent) from the slot-machine repo root.

### unroll-state-scheduler

Applies block-height-scheduled price changes from the price schedule CSV on-chain.

```bash
cargo run -- xchandles unroll-state-scheduler \
  --launcher-id <hex> \
  [--testnet11] \
  [--local] \
  [--fee 0.0025]
```

---

## verify-deployment

Validates an on-chain deployment against trusted CSV files.

```bash
cargo run -- xchandles verify-deployment \
  --launcher-id <hex> \
  [--testnet11]
```

Checks:

* Registry launcher solution and initial state
* Premine handles match `xchandles_premine_testnet11.csv` (or mainnet CSV)
* Price schedule matches `xchandles_price_schedule_testnet11.csv` (or mainnet CSV)
* Price singleton multisig configuration

---

## Register

Two-phase registration: creates a precommit coin, then (after `relative_block_height` blocks) spends the registry to claim the handle.

```bash
cargo run -- xchandles register \
  --launcher-id <hex> \
  --handle <name> \
  --nft <nft1...> \
  --payment-asset-id <hex> \
  --payment-cat-base-price <amount> \
  [--num-periods 1] \
  [--refund-address <address>] \
  [--secret <hex>] \
  [--start-time <unix>] \
  [--refund] \
  [--registration-period 31557600] \
  [--testnet11] \
  [--local] \
  [--log] \
  [--fee 0.0025]
```

**Phase 1** (first run): creates the precommit coin. The CLI prints a follow-up command.

**Phase 2** (after block delay): completes registration with the printed command.

**Refund path**: add `--refund` to claw back a precommit coin via the refund action (e.g., wrong payment, handle already taken).

---

## extend

Extends a handle's registration period in a single transaction.

```bash
cargo run -- xchandles extend \
  --launcher-id <hex> \
  --handle <name> \
  --payment-asset-id <hex> \
  --payment-cat-base-price <amount> \
  [--num-periods 1] \
  [--registration-period 31557600] \
  [--testnet11] \
  [--local] \
  [--fee 0.0025]
```

---

## initiate-update

Starts a two-phase handle transfer to a new NFT.

```bash
cargo run -- xchandles initiate-update \
  --launcher-id <hex> \
  --handle <name> \
  --new-nft <nft1...> \
  [--min-height <block>] \
  [--testnet11] \
  [--local] \
  [--fee 0.0025]
```

`--min-height` defaults to current peak + 32 blocks.

---

## execute-update

Completes a previously initiated handle update after the minimum block height.

```bash
cargo run -- xchandles execute-update \
  --launcher-id <hex> \
  --handle <name> \
  --new-nft <nft1...> \
  [--testnet11] \
  [--local] \
  [--fee 0.0025]
```

---

## expire

Re-registers an expired handle via the expiry auction pricing puzzle.

```bash
cargo run -- xchandles expire \
  --launcher-id <hex> \
  --handle <name> \
  --nft <nft1...> \
  --payment-asset-id <hex> \
  --payment-cat-base-price <amount> \
  [--expire-time <unix>] \
  [--num-periods 1] \
  [--refund-address <address>] \
  [--secret <hex>] \
  [--refund] \
  [--committed-expiration <unix>] \
  [--registration-period 31557600] \
  [--testnet11] \
  [--local] \
  [--fee 0.0025]
```

Uses the same two-phase precommit pattern as register.

---

## listen

Runs a local neighbor API and syncs registry spends.

```bash
cargo run -- xchandles listen \
  --launcher-ids <id1>,<id2> \
  [--testnet11]
```

Serves `http://localhost:3000` with a `GET /neighbors?launcher_id=&handle_hash=` endpoint (same contract as `https://api.xchandles.com`).

---

## view

Prints current registry state.

```bash
cargo run -- xchandles view \
  --launcher-id <hex> \
  [--payment-asset-id <hex>] \
  [--payment-cat-base-price <amount>] \
  [--registration-period 31557600] \
  [--testnet11]
```

Payment hints are optional overrides for price display.

---

## sign-state-update

A price singleton signer produces a partial signature for a proposed registry state change.

```bash
cargo run -- xchandles sign-state-update \
  --launcher-id <hex> \
  --new-payment-asset-id <hex> \
  --new-payment-cat-base-price <amount> \
  [--new-registration-period 31557600] \
  --my-pubkey <hex> \
  --multisig-launcher-id <hex> \
  [--payment-asset-id <hex>] \
  [--payment-cat-base-price <amount>] \
  [--registration-period 31557600] \
  [--testnet11] \
  [--debug]
```

---

## broadcast-state-update

Collects multisig signatures and broadcasts the state update.

```bash
cargo run -- xchandles broadcast-state-update \
  --launcher-id <hex> \
  --new-payment-asset-id <hex> \
  --new-payment-cat-base-price <amount> \
  [--new-registration-period 31557600] \
  --multisig-launcher-id <hex> \
  --signatures <hex,...> \
  [--payment-asset-id <hex>] \
  [--payment-cat-base-price <amount>] \
  [--registration-period 31557600] \
  [--testnet11] \
  [--fee 0.0025]
```

See the [CATalog delegated state action docs](https://docs.catalog.cat/technical-manual/other-useful-concepts#the-delegated-state-action).
