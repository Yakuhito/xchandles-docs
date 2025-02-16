---
description: Current epoch rocks - let's add some rewards to it
---

# The Add Incentives Action

_Note_: The add incentives action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/dig/add_incentives.clsp).

This action simply adds incentives to the current round. The new rewards will be continuously distributed in a linear fashion from `last_update` until `epoch_end`. They cannot be clawed back.

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the add incentives action, the announcement prefix, `a`, is concatenated to `(sha256tree (amount . epoch_end))`.

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
