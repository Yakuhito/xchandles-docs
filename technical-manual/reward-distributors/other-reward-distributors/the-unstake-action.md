---
description: I would like that NFT back, please.
---

# The Unstake Action

_Note_: The unstake action code can be found [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward_distributor/staking/unstake.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/staking/unstake.clsp)). The code for unlocking puzzles can be found [here](https://github.com/DIG-Network/reward-distributor-clsp/tree/main/rue-puzzles/actions/reward_distributor/staking/unlocking_puzzles) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/tree/main/puzzles/actions/reward_distributor/staking/unlocking_puzzles)).

This action removes one or more entries from the active entries list of the reward distributor while returning the corresponding locked asset to its owner. Because slots may be consolidated for the same payout puzzle hash (i.e., represent more than one locked asset) and this action has code shared between all non-manager types of reward distributors, an unlocking puzzle is used to give the unstake action specificity. The unlocking puzzle returns the number of shares the unstake operation should remove for the given slot, as well as a list of conditions that assert the owner is trying to unstake (usually via messages) and unlock the assets (usually by sending a message containing an appropriate delegated puzzle to the locked coins).

The unstake action triggers a payout for the whole slot, ensuring it is synced before entries are removed. There are only two unlocking puzzles, one for NFTs and one for CATs. Each spends the matching deposit slot - `(custody_puzzle_hash, shares, launcher_id or cat_amount)` - and sends a delegated puzzle to the locked coin at `my_p2`. The CAT unlocker requires `cat_shares > 0`. The NFT unlocker requires `nft_shares >= 0`. The generic unstake action only checks that `shares_to_remove` is at most the entry slot's shares. Extra unlock behavior (e.g., a lockup period) would need new locking and unlocking puzzles; the stake and unstake actions could remain unchanged.

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Aug 28th, 2026._
