---
description: This is your new owner
---

# The Update Action

_Note_: The update action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/update.clsp).

This action handles owner-specific update abilities for every entry. Namely, owners can transfer ownership of their entry or set it to resolve to another singleton. To do so, the owner singleton must create a puzzle-puzzle announcement with the contents of `(sha256tree (c handle_hash (c new_owner_launcher_id new_resolved_data)))`.

This puzzle does not create a puzzle announcement, as the owner can be assured the update happens only if the message sent to the singleton is consumed.&#x20;

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
