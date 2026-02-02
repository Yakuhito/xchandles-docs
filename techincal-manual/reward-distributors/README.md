---
description: Efficient continuous payouts
---

# Reward Distributors

First detailed in [DIP-0002](https://github.com/DIG-Network/DIPS/blob/main/DIPs/dip-0002.md) and later in [CHIP-0051](https://github.com/Chia-Network/chips/pull/165), the Reward Distributor can be seen as an efficient way to distribute rewards at a constant, per-epoch rate among a dynamic group of participants. Reward distributors come in three main flavors.

| Type    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Managed | A manager sets and removes entries with varying weights. In DIG's use, the manager sets and removes active mirrors, each with a corresponding number of shares.  During 1-week epochs, rewards are continuously allocated to active mirrors based on their shares.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| NFT     | This mode allows users to stake NFTs to earn rewards. **Collection NFT Reward Distributors** allow any NFT minted by a particular DID to be staked, assigning a weight of one to each. **Curated NFT Reward Distributors** operate based on a whitelist containing the launcher ids of stakeable NFTs, as well as their distribution weight. Such reward distributors may be **refreshable** (if the NFT's weight on the whitelist is changed, anyone may restake it to make the NFT earn rewards proportional to its new weight) or **non-refreshable** (an NFT's weight is set when the NFT is staked and cannot be updated unless the NFT is unstaked by its owner and staked again). For all NFT distributors, owners may unstake their NFTs at any time, reclaiming the NFT while stopping the accumulation of rewards. |
| CAT     | Similar to NFT types, CAT Reward Distributors allow anyone owning a particular CAT to stake it to earn proportional rewards. Owners may, at any time, unstake their CATs, which stops reward accumulation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

In all variations of the distributor, anyone can commit funds to be distributed during current and future epochs. For future epochs, funds may also be clawed back - however, a fee will be incurred (to disincentivized 'fake' commitments). This ensures that, should a manager misbehave (i.e., falsely report one or more mirror statuses on-chain), farm incentive providers have a way of stopping the majority of manager fees and changing to another farm. Manager fees are proportional to committed funds and are paid out at the start of each epoch. Payouts may be initiated by anyone or solely by the recipient via a message - this behavior is set at distributor launch and may not be changed.

The DIG Reward Distributor (manager mode) was mainly designed by [Michael Taylor ](https://github.com/MichaelTaylor3D)and implemented in Chialisp by [yakuhito](https://github.com/yakuhito). The NFT mode (DID/collection) was later added for community use. The first UI, [rewards.fireacademy.io](https://rewards.fireacademy.io/), powered the alpha deployment of the first version of distributors, allowing [DataLayer Minions](https://mintgarden.io/collections/datalayer-minions-col1k86fjeaje70hhy46yp4c2jfuhddlt66zcd6mw86zzy2d4egage3sa302t4) NFT holders to stake their minions for DIG tokens. A [precision issue](https://blog.fireacademy.io/p/too-discrete-to-handle-how-low-cat) was identified as a result of the alpha. The bug was fixed with the introduction of the latest (V2) version of the distributor, which also added the other two modes, allowing curated NFT and CAT reward distributors.

### State

Unlike XCHandles and CATalog, the DIG Reward Distributor holds a complex state that may not be arbitrarily updated by an external singleton. Instead, each action can update the dynamic state according to well-defined rules. The state of the reward distributor is `(c total_reserves (c active_shares (c round_reward_info round_time_info)))` . The first argument is used to keep track of all the funds the DIG Reward Distributor holds. The second argument tracks the number of currently active shares, which is important for reward calculation. `round_reward_info`  is defined as `(cumulative_payout . remaining_rewards)`  and tracks the cumulative payout per share since launch and the remaining amount of rewards in the current epoch, respectively. Lastly, `round_time_info`  is defined as `(last_update . epoch_end)`  and tracks the time when the reward distribution (cumulative payout) was last calculated, as well as the timestamp when the epoch ends. In Rue, the state is defined as follows:

```rs
export struct RewardInfo {
    cumulative_payout: Int,
    ...remaining_rewards: Int,
}

export struct RoundTimeInfo {
    last_update: Int,
    ...epoch_end: Int,
}

export struct RewardDistributorState {
    total_reserves: Int,
    active_shares: Int,
    reward_info: RewardInfo,
    ...round_time_info: RoundTimeInfo,
}
```

### Reserve

Another difference when compared to XCHandles/CATalog is that the DIG Reward Distributor manages funds. Specifically, a CAT determined at launch is used for rewards. This means that the reward distributor uses a single-reserve finalizer, as described [here](https://docs.catalog.cat/technical-manual/action-layer). The finalizer automatically detects output conditions starting with the special opcode `-42`and makes the reserve output them instead of the singleton - this is useful for facilitating payouts. Moreover, the finalzier automatically re-creates the reserve, ensuring it stores an amount of `total_reserves`  mojos (taken from the latest state).

### Slots

The DIG reward distributor is the first app to use different types of slots. As mentioned [here](https://docs.catalog.cat/technical-manual/slots), differentiating slots can be done by wrapping the 1st curried puzzle by a nonce layer and using different nonces for different kinds of slots. The slots of a reward distributor are defined as follows:

* **Nonce 1**: Reward slots. These hold `(epoch_start next_epoch_initialized . rewards)`. The epoch start timestamp is considered the slot's 'key' and should be unique. The second argument is 1 if the next epoch has been initialized - i.e., a slot exists with a key of `epoch_start + EPOCH_SECONDS`. The last value in the slot keeps track of the total amount of rewards committed to the epoch, including the manager fee (which will be deducted when the epoch starts). Note that a slot will no longer be accurate when the epoch starts, as anyone can add rewards to the current epoch (which does not modify 'rewards'). Moreover, undistributed rewards from previous epochs (i.e., 'change' that cannot be distributed to mirrors) will also be added to the rewards of the epoch. For example, if there are 7 shares, it's possible that a change of 5 mojos might exist, which will be 'carried over' the next epoch. Lastly, the manager will be paid out \~x% of the rewards, where x is a constant defined at reward distributor launch.
* **Nonce 2**: Commitment slots. These hold `(epoch_start clawback_ph  . rewards)`and function as 'tickets' that allow users that committed funds to a future epoch to claw part of their funds back.
* **Nonce 3**: Entry slots. These hold `(payout_puzzle_hash initial_cumulative_payout . shares)` . The first argument identifies an entry (e.g., mirror) by its payout puzzle hash. Shares describe how much an entry should be rewarded (approx. `shares / total_shares * rewards_per_second` for every second). The cumulative payout describes how much a theoretical entry holding 1 share that has been active since registry launch should be paid. By calculating the difference between the current value and the value when the entry was added, we can say how much a share needs to be paid. Since all shares are paid equally, multiplying the difference by the amount of shares yields the payout amount for a given recipient. In V2, stake/unstake puzzles allow stakes to be consolidated, allowing multiple staked entries (NFTs/CATs) to be 'stored' inside a single entry slot - thus ensuring payouts only use one coin.

Note that neither nonce 0/None (i.e., no nonce) are used for the Reward Distributor.

### Available Actions

For non-manager modes, the [Add Entry](managed-reward-distributor/the-add-entry-action.md) and [Remove Entry](managed-reward-distributor/the-remove-entry-action.md) actions are replaced by [Stake](nft-reward-distributor/the-stake-action.md) and [Unstake](nft-reward-distributor/the-unstake-action.md) actions, each containing a specific locking puzzles (as detailed on the stake action page). Moreover, the [Initiate Payout](the-initiate-payout-action.md) action can be set to one of two puzzles, depending on whether the recipient needs to approve a payment or not.&#x20;

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
