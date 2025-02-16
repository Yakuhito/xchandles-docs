---
description: Where it all starts
---

# The Register Action

_Note_: The register action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/register.clsp).

To create an entry in XCHandles, users have to register a handle. While only holding a key inside a slot (and associated neighbors) works for CATalog, XCHandles requires a more complex value structure: `(value (left_value . right_value) expiration owner_launcher_id . resolved_launcher_id)` . The values in a slot have the following meaning:

* **value**: the `sha256tree` of a handle, which can also be computed as `(sha256 1 handle)`
* **left\_value** & **right\_value**: also known as the neighbor struct (or cons-box, to be more clvm-correct), these values 'point to' the left and right neighbors in the doubly-linked list. The rule that enforces uniqueness for handles is simply **left\_value** < **right\_value**
* **expiration**: timestamp after which the handle is considered expired
* **owner\_launcher\_id**: the launcher id of the singleton that can update the entry (slot values). More precisely, the owner can freely update the **owner\_launcher\_id** of a slot (i.e., transfer a handle to a new owner) and set the **resolved\_launcher\_id** (i.e., change what the handle is resolved to)
* **resolved\_launcher\_id**: the launcher id of the singleton that the handle resolves to. This will usually be the same as **owner\_launcher\_id** and point to an NFT with an 'owner-can-update-any-field' metadata updater, but separating the two allows for more flexibility (in custody, for example). Moreover, resolved launcher ids may be used to point to different kinds of singletons in the future, such as sub-registries.

The register action behaves very similarly to the [CATalog register action](https://docs.catalog.cat/technical-manual/catalog/the-register-action), with the mention that it does NOT launch a singleton to represent the name. Instead, it is assumed that the resolved launcher id used during registration corresponds to a valid name NFT that was launched separately. Uniqueness prelaunchers are not used, as names can expire but their associated NFTs cannot be clawed back (and creating a uniqueness prelauncher for the same handle twice defeats its purpose).

The precommitment coin for XCHandles also holds more values. The reveal of its overall value will be `(c refund_info_hash (c (c secret handle) (c start_time (c new_owner_launcher_id new_resolved_launcher_id))))))` where:

* **refund\_info\_hash** is the tree hash of `(c (c cat_maker_reveal cat_maker_solution) (c pricing_puzzle_reveal pricing_solution))` . These are confirmed to correspond to those of the current coin - otherwise, the commitment coin is considered invalid and can be clawed back using the refund action.
* **secret** is used so that the handle registration is impossible to guess or brute-force before the transaction that claims the handle is sent to the mempool
* **handle** is the string representing the handle being registered (which will be validated)
* **start\_time** is a timestamp in the past that represents the start time of the handle registration
* **new\_owner\_launcher\_id** & **new\_resolved\_launcher\_id** are self-explanatory and correspond to the new handle (i.e., the one being registered)

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
