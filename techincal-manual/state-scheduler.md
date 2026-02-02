---
description: Deterministic sate changes
---

# State Scheduler

_Note_: The state scheduler puzzle can be found [here](https://github.com/Yakuhito/slot-machine/blob/master/rue-puzzles/singleton/state_scheduler.rue) ([Chialisp](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/singleton/state_scheduler.clsp)).

If you're launching a handle registry, one thing to copy from the DNS world is the initial handle auction. During the launch period, handles should be more expensive than normal, with price decreasing until it reaches the terminal ('normal') value. This ensures that people that really want a handle have a chance to register it by paying a premium, while also discouraging a single actor coming in and registering all 'rare' handles.

The state scheduler is a small puzzle intended to exist inside the price singleton of the registry during the launch period. It simply sends a message to the registry, making it update its state to a new one. `INNER_PUZZLE` is used to generate the rest of the coin's conditions - usually, these are a re-creation condition (`CREATE_COIN`) and one that asserts a block height has passed. During the launch period, the price controller singleton is expected to contain a series of state schedulers, with the last one giving control to the 'terminal' inner puzzle hash (e.g., a multisig).

Note that singletons with the state scheduler code can be spent by anyone. Because the price is expected to go down with every update, it is in the user's best interest to update the state before registering a new handle (assuming that's a possibility).

When the price singleton's first generations will contain state schedulers, the `kv_list` argument of the singleton launcher will hint it. Particularly, the format for the variable (defined [here](https://github.com/Yakuhito/slot-machine/blob/4495af3cff6b0bf44cc96acf0b5a72db4e0430a7/src/primitives/state_scheduler_info.rs#L162)) is `(price_singleton_launcher_id registry_singleton_id final_puzzle_hash state_schedule . final_puzzle_memos)`, where the final puzzle hash specifies the 'custody' inner puzzle hash the price singleton will eventually have and the state schedule is a cons-box list where each item's first value is the relative block height and the second is the updated state. The last argument can be used to describe the final puzzle in more detail (it may contain multisig data, for example).

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 21st, 2025._
