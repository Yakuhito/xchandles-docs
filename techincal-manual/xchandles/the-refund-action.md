---
description: Where registrations gone wrong are fixed
---

# The Refund Action

_Note_: The refund action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/refund.clsp).

This action is very similar to the [CATalog refund action](https://docs.catalog.cat/technical-manual/catalog/the-refund-action), except that the pricing puzzle might either be a 'normal' or 'expiry auction' one. A precommit coin can be clawed back in the following cases:

* The CAT maker puzzle hash has changed
* The payment CAT amount is wrong
* The pricing puzzle in used has changed
* The handle has already been registered

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
