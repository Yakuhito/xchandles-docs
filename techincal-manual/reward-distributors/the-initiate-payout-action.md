---
description: I know the tokens are mine - I would like them now, please. ~ Entry
---

# The Initiate Payout Action

_Note_: The initiate payout action code be found [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward-distributor/initiate_payout_with_approval.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/initiate_payout_with_approval.clsp)) (with approval) & [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward-distributor/initiate_pa.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/initiate_payout_without_approval.clsp)) (without approval).

This action initiates a payout without requiring an entry to be removed (or assets to be unstaked). Its behavior may be changed at setup, where the deployer determines whether the payout requires approval by the payout/custody puzzle hash or not. If there is no approval needed, the initiate payout action is public, and can be called by anyone to trigger a payout transaction to an entry's puzzle hash. If an approval is needed, the entry puzzle hash must send a specific message to the reward distributor to claim the payout.

To prevent wallet dusting, the accumulated amount must be at least `PAYOUT_THRESHOLD`, which is set at reward distributor launch.

While an entry's rewards are calculated using `cat_unit * PRECISION`, payouts can only be done in CAT units. For every payout, the difference between `cat_payout_amount * PRECISION` (the amount paid out in CAT units expressed with `PRECISION`) and the entry's total amount to be paid out, which is guaranteed to correspond to less than a mojo, is set to be distributed to everyone for the remainder of the period - in some sense, the payout initiator frees the entry's claim to that amount.

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the initiate payout action, the announcement prefix, `p`, is concatenated to `(sha256tree (c entry_payout_puzzle_hash entry_payout_amount))` .

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
