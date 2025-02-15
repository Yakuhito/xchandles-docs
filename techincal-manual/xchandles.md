---
description: A decentralized address book for the Chia blockchain
---

# XCHandles

From a technical standpoint, XCHandles is, at its core, a `HashMap<bytes32, bytes>`  that transforms the hash of a handle (i.e., `sha256tree`  of the handle string) to a series of bytes, usually representing a singleton launcher id. Entries usually point to a name NFT launcher id, which contains details such as 'receiver address' in its metadata (i.e., the address wallets should send assets to if the handle is used in the address field). Each entry can only be updated or transferred by its owner. Entries also have expiration dates - after registration, anyone may extend the validity of a handle. After the handle expires, it enters an auction process handled by another puzzle (set to the 'exponential premium' puzzle at launch).

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
