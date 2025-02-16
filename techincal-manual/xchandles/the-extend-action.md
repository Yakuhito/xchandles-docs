---
description: This handle is awesome - may I please have it for longer?
---

# The Extend Action

_Note_: The refund action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/extend.clsp).

Most users will want to extend the time their handle registrations. Extending is simpler than a registration because the slot has already been creating, meaning some validations (e.g., left vs. right neighbor) can be skipped. Moreover, extending doesn't require a precommitment, as the handle is already known and belongs to a known owner - meaning the process can be compressed to one transaction using offer announcements to assert payment.

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the extend action, the announcement prefix, `e`, is concatenated to `(sha256tree (total_price . handle)` .

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
