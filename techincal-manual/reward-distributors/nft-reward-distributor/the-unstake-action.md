---
description: I would like that NFT back, please.
---

# The Unstake Action

_Note_: The unstake action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/reward_distributor/nft/unstake.clsp).

This action remove an entry to the active entries of the reward distributor while returning the corresponding locked NFT to its owner.

The action does not create a puzzle announcement. Instead, the custody puzzle must send a puzzle-puzzle (18) message to the Reward Distributor with the NFT's launcher id as the contents.

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Jul 2nd, 2025._
