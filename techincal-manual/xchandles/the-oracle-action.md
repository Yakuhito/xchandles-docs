---
description: This oracle sees things that are currently true on-chain, not the future...
---

# The Oracle Action

_Note_: The oracle action code be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/actions/xchandles/oracle.clsp).

CATalog registrations are forever, which allows one to use uniqueness prelauncher to allow quick on-chain and off-chain validation that an NFT indeed represents the entry for a given CAT. XCHandles, on the other hand, has expiring handles - everyone is free to set the resolved launcher id to any singleton, and entries can expire. The oracle is a very simple action that will announce the value of a given slot on-chain. The announcement prefix, `o`, is concatenated with the slot's value hash. It's simple as that!

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
