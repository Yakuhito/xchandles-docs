---
description: Welcome, fren
---

# The Add Entry Action

_Note_: The add entry action code be found [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward-distributor/manager/add_entry.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/manager/add_entry.clsp)).

This action adds an entry to the active entries of the reward distributor. The entry immediately starts earning rewards in the active epoch.

The action does not create a puzzle announcement. Instead, the manager must send a puzzle-puzzle (18) message to the Reward Distributor with the contents of `a`  + `(sha265tree (entry_payout_puzzle_hash . entry_shares))`.

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
