---
description: 'NFT: locked.'
---

# The Stake Action

_Note_: The stake action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/reward_distributor/nft/stake.clsp).

This action adds an entry to the active entries of the reward distributor. The entry immediately starts earning rewards in the active epoch, and has a corresponding share of 1.

The action asserts that an NFT is locked with the `p2_singleton` inner puzzle by using a settlement payments announcement.  To prove the NFT belongs to a collection, a proof is used to calculate the launcher id starting with the minter DID inner puzzle hash (and going through all the intermediary coins).

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the stake action, the announcement prefix, `s`, is concatenated to the offer announcement hash (i.e., the settlements payments announcement is 'reflected' by the reward distributor).

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Jul 2nd, 2025._
