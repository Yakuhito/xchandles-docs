---
description: This will be your new owner
---

# The Initiate Update Action

_Note_: The initiate update action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/rue-puzzles/actions/xchandles/initiate_update.rue) ([Chialisp](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/initiate_update.clsp)).

This action handles owner-specific update abilities for every entry. Namely, owners can transfer ownership of their entry or set it to resolve to another singleton. To do so, the owner singleton must create a coin-puzzle message with the contents of `(sha256tree (c handle_hash (c new_owner_launcher_id new_resolved_data)))`.

To prevent certain attacks, an update cannot be completed instantly. Rather, the owner must initiate the update, wait for a 'safe' number of blocks (32 - enough to make blockchain re-orgs extremely unlikely), and then complete the update by having the child of the coin that initiated the update send another message to the distributor.

The singleton keeps track of initiated updates via update slots. An update slot's contents are `(c (c handle_hash (c new_owner_launcher_id new_resolved_data)) (c owner_initiator_coin_id min_execution_height))`.

This puzzle does not create a puzzle announcement, as the owner can be assured the update is initiated only if the message sent to the singleton is consumed. An initiated updated is canceled simply by spending the child of the owner singleton coin that initiated it.

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
