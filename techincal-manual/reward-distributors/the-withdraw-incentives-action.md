---
description: I'll take my funds elsewhere
---

# The Withdraw Incentives Action

_Note_: The withdraw incentives action code be found [here](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/rue-puzzles/actions/reward-distributor/withdraw_incentives.rue) ([Chialisp](https://github.com/DIG-Network/reward-distributor-clsp/blob/main/puzzles/actions/reward_distributor/withdraw_incentives.clsp)).

Users that committed incentives to a future epoch can call this action to 'claw back' their commitment. This is done by sending a message from the `clawback_ph` specified when the rewards were committed, which is assumed to be a custody puzzle (hence, the refund will be sent to the same puzzle).

The action does not create a puzzle announcement. Instead, the user must send a puzzle-puzzle (18) message to the Reward Distributor with no contents. If the user has multiple commitments, they should assert that the correct commitment slot is spent - commitment slots are only spent when commitments are clawed back.

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 16th, 2025._
