---
description: Welcome, fren
---

# The Add Mirror Action

_Note_: The add mirror action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/dig/add_mirror.clsp).

This action adds a mirror to the active mirrors of the reward distributor. The mirror immediately starts earning rewards in the active epoch.

The action does not create a puzzle announcement. Instead, the validator must send a puzzle-puzzle (18) message to the DIG Reward Distributor with the contents of `a`  + `(sha265tree (mirror_payout_puzzle_hash . mirror_shares))`.

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
