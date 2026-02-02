---
description: 'NFT/CAT: locked.'
---

# The Stake Action

_Note_: The stake action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/reward_distributor/nft/stake.clsp) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/staking/stake.clsp)). The code for locking puzzles can be found [here](https://github.com/DIG-Network/reward-distributor-clsp/tree/main/rue-puzzles/actions/reward-distributor/staking/locking_puzzles) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/tree/main/puzzles/actions/reward_distributor/staking/locking_puzzles)).

This action adds an entry to the active entries of the reward distributor. The entry immediately starts earning rewards in the active epoch. A general locking puzzle asserts the appropriate asset with the appropriate amount has been deposited to the distributor (i.e., becomes locked with the `p2_singleton` inner puzzle) by using a settlement payments announcement.

In general, a locking puzzle returns the number of newly locked shares and a list of conditions, which will become part of the condition list returned by the action. The stake action also allows slot consolidation: if the user already has an entry slot, they can specify it so all their shares are represented by a single slot instead of multiple ones. This means that, at payout, accumulated rewards are paid out in a single coin.

There are three locking puzzles for the reward distributor, detailed below.

### NFTs From DID

This locking puzzle is used for collection NFT distributors. To prove the NFT belongs to a collection, a proof is used to calculate the launcher id starting with the minter DID inner puzzle hash (and going through all the intermediary coins). The NFT with the resulting launcher id is then checked to be sent to a singleton-controlled inner puzzle. With this locking puzzle, every locked NFT is worth exactly one share - in other words, no NFT in the collection is allocated more than any other NFT from the collection in a given second.

Because collections are usually part of the off-chain metadata instead of on-chain attributes, this locking puzzle actually proves NFTs have been minted by a particular DID. The puzzle assumes that the given DID only minted ONE collection - any NFT minted by the DID set at launch can be staked, which means the DID owner needs to be trusted not to maliciously mint many NFTs and stake all of them to collect future rewards. In general, while this category of NFT distributors is more trustless than managed reward distributors, it may still require trust. The next locking puzzle, for example, assumes the DataStore is operated as expected and not compromised. Lastly, the CAT reward distributor trusts the CAT's TAIL and its issuer (should they have the ability to increase the CAT supply).

### NFTs From DL

DL stands for DataLayer. Curated NFT reward distributors allow any NFT on a whitelist to be staked, with the weight of a given NFT also being specified on the said list. The list is stored in a Merkle tree inside a DataLayer store, with the trusted launcher id being set at distributor launch. If the weight of the NFT in the list is changed by the store owner, the NFT may or may not be restaked via the [Refresh Action](the-refresh-action.md).

### CAT

As the name suggests, the CAT locking puzzle allows a CAT with a certain asset id to be locked in exchange for a number of shares equal to its amount.



_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Jul 2nd, 2025._
