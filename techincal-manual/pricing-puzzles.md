---
description: '"That will be 7 XCH, sir."'
---

# Pricing Puzzles

Different registries might have different pricing rules - one registry might to ask for the same amount of payment CATs for any handle, while others might implement dynamic pricing based on custom rules. To accommodate this need, XCHandles was build to have two puzzles: one for determining the price of a handle during registration (and renewal/registration extension), and one for determining the price of an expired handle (during an 'expiry auction').

The way pricing puzzles work was mainly informed by observations from Ethereum Name Service's history. For example, the pricing strategy of a registry might need to change during its lifetime as the registry controllers discovers new data about what's working and what's not. For example, ENS experimented with 2 different ways to handle name expiration between implementing the reverse exponential auction (see 'Exponential Premium' for more details on the technique). This led to the decision to include the two puzzles in the registry's state rather than making them curried in argument. This means that the price singleton, aside from setting a new [CAT Maker](https://docs.catalog.cat/technical-manual/other-useful-concepts#cat-makers), can also set new pricing strategies. This functionality can always be opted out of by melting the price singleton (or by using a custom inner puzzle that restricts what state updates are allowed).

### _Factor Pricing_

_Note_: The factor pricing puzzle can be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/default_puzzles/factor_pricing.clsp).

This pricing strategy allows handles to be priced similar to [ENS domains](https://docs.ens.domains/faq#what-does-it-cost-to-register-a-eth-domain). The price is determined as the base price (a curried in parameter) multiplied by a factor which depends on two variables: the handle's length (shorter handles will be more expensive) and whether the handle contains numbers (handles containing numbers cost half of what their only-characters counterparts do). The table below describes the prices for handle registration/renewal **per year** (to get the price for three years, for example, one would have to multiply the values in the table by 3 - there is no discount).

<table><thead><tr><th width="100">Length</th><th width="130" data-type="checkbox">Has Numbers?</th><th width="100">Factor</th><th width="173">Example Handle</th><th>Example Cost</th></tr></thead><tbody><tr><td>3</td><td>false</td><td>128</td><td>@abc</td><td>640 wUSDC.b per year</td></tr><tr><td>3</td><td>true</td><td>64</td><td>@ab1</td><td>320 wUSDC.b per year</td></tr><tr><td>4</td><td>false</td><td>64</td><td>@abcd</td><td>320 wUSDC.b per year</td></tr><tr><td>4</td><td>true</td><td>32</td><td>@abc1</td><td>160 wUSDC.b per year</td></tr><tr><td>5</td><td>false</td><td>16</td><td>@abcde</td><td>80 wUSDC.b per year</td></tr><tr><td>5</td><td>true</td><td>8</td><td>@a1234</td><td>40 wUSDC.b per year</td></tr><tr><td>6+</td><td>false</td><td>2</td><td>@example</td><td>10 wUSDC.b per year</td></tr><tr><td>6+</td><td>true</td><td>1</td><td>@example1</td><td>5 wUSDC.b per year</td></tr></tbody></table>

The table assumes a **base price** of 5.00 wUSDC.b (5000 when translated to CAT mojos). Note that the payment token is determined by the CAT maker - wUSDC.b is only used here to offer meaning to the example.

### _Exponential Premium_

_Note_: The exponential premium puzzle can be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/default_puzzles/exponential_premium.clsp).

After a handle expires, determining a good 'next owner' is a difficult problem to solve. A clever solution is currently being used by ENS: whoever wants the handle the most would be willing to pay more for it, so auctioning the handle is a good way to go. The price starts at a high value and then decreases each second - giving everyone the chance to register it at the price they consider 'fair.' Linear pricing (e.g., the price decreases $5/hour) is unlikely to work because starting at a price no one would be willing to pay (e.g., $100 million) would result in an auction that is either too long or too fast (the price decreases very rapidly). Instead, XCHandles implements an 'exponential premium' pricing strategy, where the auction starts with a very high value and the premium halves every 28 hours. The table below shows how the premium evolves over a 28-day auction when it starts at 100,000,000 wUSDC.b:&#x20;

| Time            | Approx. Premium     |
| --------------- | ------------------- |
| Day 0, Hour 0   | 99999999.63 wUSDC.b |
| Day 0, Hour 1   | 97153878.78 wUSDC.b |
| Day 0, Hour 12  | 70710677.75 wUSDC.b |
| Day 1, Hour 0   | 49999999.63 wUSDC.b |
| Day 1, Hour 12  | 35355338.69 wUSDC.b |
| Day 2, Hour 0   | 24999999.63 wUSDC.b |
| Day 3, Hour 0   | 12499999.63 wUSDC.b |
| Day 7, Hour 0   | 781249.628 wUSDC.b  |
| Day 14, Hour 0  | 6103.143 wUSDC.b    |
| Day 21, Hour 0  | 47.311 wUSDC.b      |
| Day 27, Hour 0  | 0.373 wUSDC.b       |
| Day 28, Hour 22 | 0.033 wUSDC.b       |
| Day 28, Hour 23 | 0.019 wUSDC.b       |
| Day 28, Hour 24 | 0.008 wUSDC.b       |

Note that the exponential premium puzzle approximates the floating value of `start_premium / 2**(time_passed /auction_start)` , with an auction taking exactly 28 days. The overall effect is that the premium halves every day, but it also takes intermediary values throughout the day. The staring premium is calculated such that the premium value after the last second is rounded down to 0. The hourly evolution of the premium can be seen in the table below:

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>Premium decreases exponentially until day 28, when it reaches 0</p></figcaption></figure>

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
