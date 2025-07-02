---
description: Someone didn't renew this awesome handle - let's get it!
---

# The Expire Action

_Note_: The expire action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/expire.clsp).

When handles expire, they're auctioned off by using a second pricing puzzle. Usually (and by default), the other pricing puzzle will use the time when the handle expired and the registration time to calculate a premium, which is added to the base price. The premium decreases (exponentially in the puzzle used at launch) as more time passes - essentially creating a public 'auction' where it's in the potential owners'  best interest to pay their maximum price for the handle to secure it.

Because the handle is changing owners, precommitment coins are used to solve the concern of potential mempool snipers (malicious farmers). The value of a precommitment coin will be the same as that for registrations. Note that the pricing puzzle will now describe the auction puzzle, not the 'standard one.'

The action also creates a puzzle announcement that be asserted to ensure the dApp singleton is running the right action with the right parameters. For the extend action, the announcement prefix, `x`, is concatenated to the precommit coin's full puzzle hash.

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
