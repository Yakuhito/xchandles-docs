---
description: Mirror is offline - remove it from the active mirror list
---

# The Remove Mirror Action

_Note_: The remove mirror action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/dig/remove_mirror.clsp).

This action is called by the validator to stop paying rewards to a particular mirror. The mirror is automatically paid out the remaining amount (i.e., the rewards the owner did not claim).

The action does not create a puzzle announcement. Instead, the validator must send a puzzle-puzzle (18) message to the DIG Reward Distributor with the contents of `r`  + `(sha265tree (mirror_payout_puzzle_hash . mirror_shares))`.

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._&#x44;O
