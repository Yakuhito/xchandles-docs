---
description: Because everyone needs NFTs with dynamic weights, for some reason
---

# The Refresh Action

_Note_: The refresh action code can be found [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward_distributor/staking/refresh_nfts_from_dl.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/staking/refresh_nfts_from_dl.clsp)).

The refresh action is the newest addition to the reward distributor repertoire. It was added to address a particular short-coming of the curated NFT distributor in some cases. Particularly, after an NFT is staked, the DataLayer store owner may update the NFT's weight. The _non-refreshable_ curated NFT reward distributor behavior is that the NFT's weight in the distributor (number of shares) is not changed until the owner unstakes the NFT. This has advantages in some cases, but not all.

The refresh action allows curated NFT reward distributors to be refreshable. Anyone can trigger this action to essentially restake a set of NFTs whose weights mismatch those in the DataLayer store. Presumably, the owner themselves would call it after the store is updated - if that is not the case, a third party observer may.

The NFT's weight lives in its deposit slot, not in a nonce on the `p2_singleton` lock. Refresh proves the new DataLayer leaf, spends the old deposit slot (`old_shares`), creates a new deposit slot (`new_shares`), and restakes the NFT still at `my_p2` - the delegated puzzle recreates the locked coin there with the same settlement hint. The share delta must be nonzero; both new and old shares must be `>= 0`. This thus allows curated NFT reward distributors to respond to updates of dynamic lists, where weights (e.g., an NFT's rarity) may be changed at any time by the store owner.

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Aug 28th, 2026._
