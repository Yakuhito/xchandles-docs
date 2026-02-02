---
description: This is your new owner
---

# The Execute Update Action

_Note_: The execute update action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/rue-puzzles/actions/xchandles/execute_update.rue) ([Chialisp](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/execute_update.clsp)).

Once an update has been [initiated](the-initiate-update-action.md) and a minimum number of blocks has passed, the owner may execute the update by calling this action. In doing so, messages must also be sent from the new singleton(s) that the handle will be owned by and point to.

This puzzle does not create a puzzle announcement, as the owner can be assured the update is executed only if the message sent to the singleton is consumed.&#x20;

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 2nd, 2026._
