---
description: Where registrations gone wrong are fixed
---

# The Refund Action

_Note_: The refund action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/rue-puzzles/actions/xchandles/refund.rue) ([Chialisp](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/refund.clsp)).

This action is very similar to the [CATalog refund action](https://docs.catalog.cat/technical-manual/catalog/the-refund-action), except that the pricing puzzle might either be a 'normal' or 'expiry auction' one. A precommit coin can be clawed back in the following cases:

* The CAT maker puzzle hash has changed
* The payment CAT amount is wrong
* The pricing puzzle in used has changed
* The handle has already been registered (and is still valid - i.e., has not expired)

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the refund action, the announcement prefix, `$` , is concatenated to the precommit coin's full puzzle hash.

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
