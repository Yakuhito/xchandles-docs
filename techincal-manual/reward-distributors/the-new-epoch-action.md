---
description: Transitions to a new reward period
---

# The New Epoch Action

_Note_: The new epoch action code be found [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward-distributor/new_epoch.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/new_epoch.clsp)).

Because reward rates might change, the Reward Distributor uses the concept of 'epochs', which are simply one-week periods. The rewards committed to each epoch may be different. Inside each round, rewards are distributed linearly (e.g., \~25% of the rewards will be distributed after 25% of the time in the round has passed).

Starting a new epoch requires a 'slot oracle' - spending and recreating a reward slot with the same value. This ensures that the correct rewards are allocated. The slot either:

* Represents the new epoch, in which case it contains the reward amount
* Represents a previous epoch but indicates that the next slot is not initialized, in which case the reward amount for the epoch to be initialized is 0

When a new epoch starts, the committed rewards, which were previously stored as CAT units (i.e., 1 = one mojo), are multiplied by PRECISION (usually `u64::MAX`), allowing a much more precise reward allocation to all entries.

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the new epoch action, the announcement prefix, `e`, is concatenated to `(sha256tree epoch_end)`.

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
