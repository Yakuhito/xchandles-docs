---
description: A decentralized address book for the Chia blockchain
---

# XCHandles

From a technical standpoint, XCHandles is, at its core, a `HashMap<bytes32, bytes32>`  that transforms the hash of a handle (i.e., `sha256tree`  of the handle string) to a singleton launcher id. Entries usually point to a name NFT launcher id, which contains details such as receiver address (resolved as the owner of the NFT), as well as other info in their metadata (e.g., display name and image). Each entry can only be updated or transferred by its owner. Entries also have expiration dates - after registration, anyone may extend the validity of a handle. After the handle expires, it enters an auction process handled by another puzzle (set to the 'exponential premium' puzzle at launch).

Before proceeding, it's highly recommended that you familiarize yourself with the primitives in the [CATalog docs](https://docs.catalog.cat/technical-manual/slots), as XCHandles uses them in more complex ways.

Note that the XCHandles registry also includes a [delegated state action](https://docs.catalog.cat/technical-manual/other-useful-concepts#the-delegated-state-action).

Generally, announcements and messages from the main registry are sent using a one-byte prefix followed by a hash of the actual message contents. To prevent collisions, the prefixes and message structures are defined in a single file ([here](https://github.com/xch-dev/chia-wallet-sdk/blob/main/crates/chia-sdk-driver/src/primitives/action_layer/xchandles_registry_prefix.rs)).

### Slots

Handle slots contain data about registered handles and use a nonce of **1**. Slots tracking initiated updates use a nonce of **2**. More information about the data held in these slots can be found under [The Register Action](the-register-action.md) and [The Initiate Update Action](the-initiate-update-action.md).



_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
