---
description: Efficient continuous payouts
---

# Reward Distributors

First detailed in [DIP-0002](https://github.com/DIG-Network/DIPS/blob/main/DIPs/dip-0002.md), the DIG Reward Distributor can be seen as a permissioned liquidity farm. A validator sets and removes active mirrors, each with a corresponding number of shares. During 1-week epoch, rewards committed by anyone are continuously allocated to active mirrors based on their shares. Payout transactions can be triggered by anyone, and payouts are automatically made when a mirror becomes offline.

Anyone can commit funds to be distributed during current and future epochs. For future epochs, funds may also be clawed back - however, a fee will be incurred (to disincentivized 'fake' commitments). This ensures that, should a validator misbehave (i.e., falsely report one or more mirror statuses on-chain), farm incentive providers have a way of stopping the majority of validator fees and changing to another farm. Validator fees are proportional to committed funds and are paid out at the start of each epoch.

The DIG Reward Distributor was mainly designed by [Michael Taylor ](https://github.com/MichaelTaylor3D)and implemented in Chialisp by [yakuhito](https://github.com/yakuhito).

### State

Unlike XCHandles and CATalog, the DIG Reward Distributor holds a complex state that may not be arbitrarily updated by an external singleton. Instead, each action can update the dynamic state according to well-defined rules. The state of the reward distributor is `(list total_reserves active_shares round_reward_info round_time_info)` . The first argument is used to keep track of all the funds the DIG Reward Distributor holds. The second argument tracks the number of currently active shares, which is important for reward calculation. `round_reward_info`  is defined as `(cumulative_payout . remaining_rewards)`  and tracks the cumulative payout per share since launch and the remaining amount of rewards in the current epoch, respectively. Lastly, `round_time_info`  is defined as `(last_update . epoch_end)`  and tracks the time when the reward distribution (cumulative payout) was last calculated, as well as the timestamp when the epoch ends.

### Reserve

Another difference when compared to XCHandles/CATalog is that the DIG Reward Distributor manages funds. Specifically, a CAT determined at launch is used for rewards. This means that the reward distributor uses a single-reserve finalizer, as described [here](https://docs.catalog.cat/technical-manual/action-layer). The finalizer automatically detects output conditions starting with the special opcode `-42`and makes the reserve output them instead of the singleton - this is useful for facilitating payouts. Moreover, the finalzier automatically re-creates the reserve, ensuring it stores an amount of `total_reserves`  mojos (taken from the latest state).

### Slots

The DIG reward distributor is the first app to use different types of slots. As mentioned [here](https://docs.catalog.cat/technical-manual/slots), differentiating slots can be done by wrapping the 1st curried puzzle by a nonce layer and using different nonces for different kinds of slots. The slots of a reward distributor are defined as follows:

* **Nonce 1**: Reward slots. These hold `(epoch_start next_epoch_initialized . rewards)`. The epoch start timestamp is considered the slot's 'key' and should be unique. The second argument is 1 if the next epoch has been initialized - i.e., a slot exists with a key of `epoch_start + EPOCH_SECONDS`. The last value in the slot keeps track of the total amount of rewards committed to the epoch, including the validator fee (which will be deducted when the epoch starts). Note that a slot will no longer be accurate when the epoch starts, as anyone can add rewards to the current epoch (which does not modify 'rewards'). Moreover, undistributed rewards from previous epochs (i.e., 'change' that cannot be distributed to mirrors) will also be added to the rewards of the epoch. For example, if there are 7 shares, it's possible that a change of 5 mojos might exist, which will be 'carried over' the next epoch. Lastly, the validator will be paid out \~x% of the rewards, where x is a constant defined at reward distributor launch.
* **Nonce 2**: Commitment slots. These hold `(epoch_start clawback_ph  . rewards)`and function as 'tickets' that allow users that committed funds to a future epoch to claw part of their funds back.
* **Nonce 3**: Entry slots. These hold `(payout_puzzle_hash initial_cumulative_payout . shares)` . The first argument identifies an entry (e.g., mirror) by its payout puzzle hash. Shares describe how much an entry should be rewarded (approx. `shares / total_shares * rewards_per_second` for every second). The cumulative payout describes how much a theoretical entry holding 1 share that has been active since registry launch should be paid. By calculating the difference between the current value and the value when the entry was added, we can say how much a share needs to be paid. Since all shares are paid equally, multiplying the difference by the amount of shares yields the payout amount for a given recipient.

Note that neither nonce 0 nor None (i.e., no nonce) are used for the Reward Distributor.

### The NFT Reward Distributor

The reward distributor was subsequently updated to also enable distributing rewards in a permissionless manner to NFT owners. More specifically, the add and remove entry actions can be swapped to stake and unstake actions. These allow people to temporarily lock in an NFT from a given collection (identified by minter DID) for a share of the distributor rewards. Each NFT is weighted equally in distribution calculation (i.e., all NFTs have 1 share each).

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
