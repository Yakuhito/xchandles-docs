---
description: A decentralized address book for the Chia blockchain
---

# XCHandles

From a technical standpoint, XCHandles is, at its core, a `HashMap<bytes32, bytes32>`  that transforms the hash of a handle (i.e., `sha256tree`  of the handle string) to a singleton launcher id. Entries usually point to a name NFT launcher id, which contains details such as receiver address (resolved as the owner of the NFT), as well as other info in their metadata (e.g., display name and image). Each entry can only be updated or transferred by its owner. Entries also have expiration dates - after registration, anyone may extend the validity of a handle. After the handle expires, it enters an auction process handled by another puzzle (set to the 'exponential premium' puzzle at launch).

Before proceeding, it's highly recommended that you familiarize yourself with the primitives in the [CATalog docs](https://docs.catalog.cat/technical-manual/slots), as XCHandles uses them in more complex ways.

Note that the XCHandles registry also includes a [delegated state action](https://docs.catalog.cat/technical-manual/other-useful-concepts#the-delegated-state-action).

Generally, announcements and messages from the main registry are sent using a one-byte prefix followed by a hash of the actual message contents. To prevent collisions, the prefixes and message structures are defined in a single file ([here](https://github.com/Yakuhito/chia-wallet-sdk/blob/main/crates/chia-sdk-driver/src/primitives/action_layer/xchandles_registry_prefix.rs)).

### Registry State

The registry singleton's inner state (`XchandlesRegistryState`) contains three puzzle hashes:

| Field | Purpose |
|-------|---------|
| `cat_maker_puzzle_hash` | Puzzle that returns the full payment CAT puzzle hash for registrations |
| `pricing_puzzle_hash` | Puzzle that quotes registration/renewal prices (and periods) |
| `expired_handle_pricing_puzzle_hash` | Puzzle that quotes post-expiry auction prices |

These are curried at launch and can be updated later via the [delegated state action](https://docs.catalog.cat/technical-manual/other-useful-concepts#the-delegated-state-action).

### Registry Constants

Immutable per-deployment values (`XchandlesConstants`):

| Field | Purpose | Default (CLI) |
|-------|---------|---------------|
| `launcher_id` | Registry singleton launcher id | Set at launch |
| `precommit_payout_puzzle_hash` | Where precommit refunds are sent | `--payout-address` at launch |
| `relative_block_height` | Blocks between precommit and register | **32** |
| `price_singleton_launcher_id` | Launcher id of the price multisig singleton | Set at launch |

### Action Merkle Tree

The registry action layer contains eight puzzles (in Merkle tree order):

1. Expire
2. Extend
3. Oracle
4. Register
5. Initiate Update
6. Execute Update
7. Refund
8. Delegated State

### Announcement and Message Prefixes

**Created announcements**:

| Prefix | Action | Payload |
|--------|--------|---------|
| `r` | Register | Precommit coin full puzzle hash |
| `$` | Refund | Precommit coin full puzzle hash |
| `e` | Extend | `sha256tree(total_price . handle)` |
| `x` | Expire | Precommit coin full puzzle hash |
| `o` | Oracle | Slot value hash |

**Received messages**:

| Prefix | Purpose | Payload |
|--------|---------|---------|
| `s` | Update state | New `XchandlesRegistryState` hash |
| `a` | Register owner | Precommit coin full puzzle hash |
| `b` | Register resolved | Precommit coin full puzzle hash |
| `e` | Expire owner | Precommit coin full puzzle hash |
| `f` | Expire resolved | Precommit coin full puzzle hash |
| `i` | Initiate update | Update slot value hash |
| `u` | Execute update (old owner) | Update slot value hash |
| `o` | Execute update (new owner) | Update slot value hash |
| `r` | Execute update (new resolved) | Update slot value hash |

### Slots

Handle slots contain data about registered handles and use a nonce of **1**. Slots tracking initiated updates use a nonce of **2**. More information about the data held in these slots can be found under [The Register Action](the-register-action.md) and [The Initiate Update Action](the-initiate-update-action.md).

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025. Updated Jul 2026._
