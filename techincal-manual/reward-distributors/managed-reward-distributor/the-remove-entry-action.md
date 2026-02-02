---
description: Mirror is offline - remove it from the active entry list
---

# The Remove Entry Action

_Note_: The remove entry action code be found [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward-distributor/manager/remove_entry.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/manager/remove_entry.clsp)).

This action is called by the manager to stop paying rewards to a particular entry. The entry is automatically paid out the remaining amount (i.e., the rewards the owner did not claim). Given the amount to be paid out is calculated in `PRECISION` units, the difference between `cat_payout_amount * PRECISION` and the total to be paid out, which is guaranteed to be lower than 1 CAT unit, is re-allocated to be redistributed in the current or future epochs.

The action does not create a puzzle announcement. Instead, the manager must send a puzzle-puzzle (18) message to the Reward Distributor with the contents of `r`  + `(sha265tree (entry_payout_puzzle_hash . entry_shares))`.

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
