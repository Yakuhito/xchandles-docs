---
description: '"I''ll probably incentivize this future round"'
---

# The Commit Incentives Action

_Note_: The commit incentives action code be found [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward-distributor/commit_incentives.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/commit_incentives.clsp)).

Unlike adding incentives, commitments are not necessarily binding. Rewards committed to a future epoch can be clawed back (minus a % fee) until the corresponding epoch starts.

This action can affect slots in two different ways. First, if the reward slot corresponding to the epoch the commitment is made for exists, the action will simply spend the slot and re-create it with higher rewards. Second, if the slot is not initialized, the user will provide the last initialized slot. In that case, the action will initialize all slots between the lastly initialized reward slot and the target one. A newly-initialized slot reward is 0 unless it's the target epoch, in which case the rewards will be set to the commitment amount.

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the commit incentives action, the announcement prefix, `c`, is concatenated to `(sha256tree (epoch_time next_epoch_time . total_rewards))` , which is also the value hash of the commitment slot that is created.&#x20;

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
