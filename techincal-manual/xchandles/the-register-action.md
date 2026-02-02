---
description: Where it all starts
---

# The Register Action

_Note_: The register action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/rue-puzzles/actions/xchandles/register.rue) ([Chialisp](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/register.clsp)).

To create an entry in XCHandles, users have to register a handle. While only holding a key inside a slot (and associated neighbors) works for CATalog, XCHandles requires a more complex value structure: `(c (c handle_hash (c left right)) (c expiration (c owner resolved)))` . In Rue, this is defined as follows:

```rs
export struct HandleSlotNeighbors {
    left_value: Bytes32,
    ...right_value: Bytes32,
}

export struct HandleSlotValueA {
    handle_hash: Bytes32,
    ...neighbors: HandleSlotNeighbors,
}

export struct HandleSlotData {
    owner_launcher_id: Bytes32,
    ...resolved_launcher_id: Bytes32,
}

export struct HandleSlotValueB {
    expiration: Int,
    ...data: HandleSlotData,
}

export struct XchandlesHandleSlotValue {
    a: HandleSlotValueA,
    ...b: HandleSlotValueB,
}
```

The values in a slot have the following meaning:

* **value**: the `sha256tree` of a handle, which can also be computed as `(sha256 1 handle)`
* **left\_value** & **right\_value**: also known as the neighbor struct (or cons-box, to be more clvm-correct), these values 'point to' the left and right neighbors in the doubly-linked list. The rule that enforces uniqueness for handles is simply **left\_value** < **right\_value**
* **expiration**: timestamp after which the handle is considered expired
* **owner**: the launcher id of the singleton that can update the entry (slot value). More precisely, the owner can freely update the **owner** of a slot (i.e., transfer a handle to a new owner) and set the **resolved** (i.e., change the singleton that the handle is resolved to)
* **resolved**: the launcher id of the singleton that the handle resolves to. This will usually be the same as **owner** and point to an NFT with an 'owner-can-update-any-field' metadata updater, but separating the two allows for more flexibility (in custody, for example).

The register action behaves very similarly to the [CATalog register action](https://docs.catalog.cat/technical-manual/catalog/the-register-action), with the mention that it does NOT launch a singleton to represent the name. Instead, it is assumed that the resolved launcher id used during registration corresponds to a valid name NFT that was launched separately. Uniqueness prelaunchers are not used, as names can expire but their associated NFTs cannot be clawed back (and creating a uniqueness prelauncher for the same handle twice defeats its purpose).

At registration, the owner and resolved singleton(s) need to send messages to the registry to confirm the user owns them.

The precommitment coin for XCHandles also holds more values. The reveal of its overall value will be the hash of `(c (c (c cat_maker_hash cat_maker_solution) (c pricing_puzzle_hash pricing_puzzle_solution)) (c (c handle secret) (c owner_launcher_id resolved_launcher_id)))` where:

* Refund info is a sub-struct, `(c (c cat_maker_hash cat_maker_solution) (c pricing_puzzle_hash pricing_solution))` . These values are confirmed to correspond to those of the current coin - otherwise, the commitment coin is considered invalid and can be clawed back using the refund action.
* **handle** is the string representing the handle being registered (which will be validated)
* **secret** is used so that the handle registration is impossible to guess or brute-force before the transaction that claims the handle is sent to the mempool
* **start\_time** is a timestamp in the past that represents the start time of the handle registration
* **owner\_launcher\_id** & **resolved\_launcher\_id** are self-explanatory and correspond to the new handle (i.e., the one being registered)

This translates to the following Rue struct:

```rs
export struct PuzzleDetails {
    puzzle_hash: Bytes32,
    ...puzzle_solution: Any,
}

export struct PrecommitValueA {
    cat_maker_details: PuzzleDetails,
    ...pricing_puzzle_details: PuzzleDetails,
}

export struct PrecommitHandleAndSecret {
    handle: String,
    ...secret: Any,
}

export struct PrecommitValueB {
    handle_and_secret: PrecommitHandleAndSecret,
    ...data: HandleSlotData,
}


export struct PrecommitValue {
    a: PrecommitValueA,
    ...b: PrecommitValueB,
}
```

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the register action, the announcement prefix, `r` , is concatenated to the precommit coin's full puzzle hash .

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
