---
description: Where the interesting math is found
---

# The Sync Action

_Note_: The sync action code be found [here](https://github.com/DIG-Network/reward-distributor-clsp/tree/main/rue-puzzles/actions/reward-distributor/sync.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/sync.clsp)).

The sync action is used to distribute rewards since the last reward distribution. The cumulative payout grows by `(/ (* remaining_rewards (- update_time last_update)) (* active_shares (- epoch_end last_update)))` . In essence, `(/ (- update_time last_update) (- epoch_end last_update))`  represents the fraction of time passed compared to the remaining time in the round. Multiplying this by the remaining rewards amount yields the rewards to be distributed. Dividing the results by the number of active shares will return the change in cumulative payout, which is always defined for one share. Note that this result is 'rounded down' by division operations, and that the 'change' is stored in the new state's `remaining_rewards`.

Syncing can only be done for the current epoch - when the epoch ends (i.e., `last_update = epoch_end`), the 'new epoch' action must be called.

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the sync action, the announcement prefix, `s`, is concatenated to `(sha256tree (update_time . epoch_end))`.

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
