---
description: I know the tokens are mine - I would like them now, please. ~ Mirror
---

# The Initiate Payout Action

_Note_: The initiate payout action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/dig/initiate_payout.clsp).

This public action can be called by anyone to trigger a payout transaction to a mirror. To prevent wallet dusting, the accumulated amount must be at least `PAYOUT_THRESHOLD`, which is set at reward distributor launch.

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the initiate payout action, the announcement prefix, `p`, is concatenated to `(sha256tree ((mirror_payout_puzzle_hash . mirror_shares) . (mirror_initial_cumulative_payout new_mirror_initial_cumulative_payout)))` .

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
