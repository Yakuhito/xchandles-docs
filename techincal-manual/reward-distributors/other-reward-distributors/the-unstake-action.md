---
description: I would like that NFT back, please.
---

# The Unstake Action

_Note_: The unstake action code be found [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward-distributor/staking/unstake.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/staking/unstake.clsp)). The code for locking puzzles can be found [here](https://github.com/DIG-Network/reward-distributor-clsp/tree/main/rue-puzzles/actions/reward-distributor/staking/unlocking_puzzles) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/tree/main/puzzles/actions/reward_distributor/staking/unlocking_puzzles)).

This action removes one or more entries from the active entries list of the reward distributor while returning the corresponding locked asset to its owner. Because slots may be consolidated for the same payout puzzle hash (i.e., represent more than one locked asset) and this action has code shared between all non-manager types of reward distributors, an unlocking puzzle is used to give the unstake action specificity. The unlocking puzzle returns the number of shares the unstake operation should remove for the given slot, as well as a list of conditions that assert the owner is trying to unstake (usually via messages) and unlock the assets (usually by sending a message containing an appropriate delegated puzzle to the locked coins).

The unstake action triggers a payout for the whole slot, ensuring it is synced before entries are removed. There are only two unlocking puzzles, one for NFTs and one for CATs. This is because an NFT - under any NFT subtype of the distributor - is locked using a nonce of `(payout_puzzle_hash . shares)`, allowing the locking puzzle to asset the NFT's shares while unlocking it. Locked CATs have the same nonce structure. Note that, should extra behavior be desired in the unlocking process (e.g., only allowing CATs to be unlocked 7 days after they are locked), a new nonce format would need to be created, and the corresponding locking and unlocking puzzles would need to be written. The stake and unstake actions could, however, remain unchanged.

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Jul 2nd, 2025._
