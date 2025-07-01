---
description: '"That will be 7 XCH, sir."'
---

# Pricing Puzzles

Different registries might have different pricing rules - one registry might to ask for the same amount of payment CATs for any handle, while others might implement dynamic pricing based on custom rules. To accommodate this need, XCHandles was build to have two puzzles: one for determining the price of a handle during registration (and renewal/registration extension), and one for determining the price of an expired handle (during an 'expiry auction').

The way pricing puzzles work was mainly informed by observations from Ethereum Name Service's history. The pricing strategy of a registry might need to change during its lifetime as the registry controllers discovers new data about what's working and what's not. For example, ENS experimented with 2 different ways to handle name expiration before implementing the reverse exponential auction (see 'Exponential Premium' for more details on the technique). This led to the decision to include the two puzzles in the registry's state rather than making them curried arguments. The pricing strategy is set by the price singleton, which can also set a new [CAT Maker](https://docs.catalog.cat/technical-manual/other-useful-concepts#cat-makers). This functionality can always be turned off by melting the price singleton (or by using a custom inner puzzle that restricts what state updates are allowed).

### _Factor Pricing_

_Note_: The factor pricing puzzle can be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/default_puzzles/factor_pricing.clsp).

This 'factor' pricing strategy allows handles to be priced similar to [ENS domains](https://docs.ens.domains/faq#what-does-it-cost-to-register-a-eth-domain). The price is determined as the base price (a curried in parameter) multiplied by a factor which depends on two variables: the handle's length (shorter handles will be more expensive) and whether the handle contains numbers (handles containing numbers cost half of what their only-characters counterparts do). The table below describes the prices for handle registration/renewal **per year** (to get the price for three years, for example, one would have to multiply the values in the table by 3 - there is no discount).

<table><thead><tr><th width="100">Length</th><th width="130" data-type="checkbox">Has Numbers?</th><th width="100">Factor</th><th width="173">Example Handle</th><th>Cost</th></tr></thead><tbody><tr><td>3</td><td>false</td><td>128</td><td>@abc</td><td>640 wUSDC.b per year</td></tr><tr><td>3</td><td>true</td><td>64</td><td>@ab1</td><td>320 wUSDC.b per year</td></tr><tr><td>4</td><td>false</td><td>64</td><td>@abcd</td><td>320 wUSDC.b per year</td></tr><tr><td>4</td><td>true</td><td>32</td><td>@abc1</td><td>160 wUSDC.b per year</td></tr><tr><td>5</td><td>false</td><td>16</td><td>@abcde</td><td>80 wUSDC.b per year</td></tr><tr><td>5</td><td>true</td><td>8</td><td>@a1234</td><td>40 wUSDC.b per year</td></tr><tr><td>6+</td><td>false</td><td>2</td><td>@example</td><td>10 wUSDC.b per year</td></tr><tr><td>6+</td><td>true</td><td>1</td><td>@example1</td><td>5 wUSDC.b per year</td></tr></tbody></table>

The table assumes a **base price** of 5.00 wUSDC.b (5000 when translated to CAT mojos). Note that the payment token is determined by the CAT maker - wUSDC.b is the token registrations are planned to be priced in after mainnet launch. One year is the normal registration period, but other subregistries may choose other periods - DIG handles, for example, plans to price registrations per week.

More generally, pricing puzzles return `(price . registration_delta_seconds)`, where `price`is the amount of payment CATs to be paid and `registration_delta_seconds` is a positive integer representing the number of seconds the registration should be extended for. Pricing puzzles have a solution that always starts with three truths:

* `Buy_Time`: this is a timestamp verified to be in the past. Not used in the factor pricing puzzle, but added so the solution format is consistent with the exponential premium puzzle solution.
* `Current_Expiration` , which is the verified current expiration of the handle whose price is being quoted. This will only be '0' if the handle is being registered. The factor pricing puzzle does not use this truth.
* `Handle` , a string containing the value being registered. The pricing puzzle is expected to perform validation for this handle. For example, the factor pricing puzzle ensures a handle is 3-31 characters long and only contains 0-9a-z characters.

The two truths may be followed by an arbitrary solution. For the factor pricing puzzle, this solution is simply `num_periods` (formerly `num_years`), which allows users to register a handle for more than 1 period at a time.

### _Exponential Premium_

_Note_: The exponential premium puzzle can be found [here](https://github.com/Yakuhito/slot-machine/blob/master/puzzles/default_puzzles/exponential_premium.clsp).

After a handle expires, determining a good 'next owner' is a difficult problem to solve. A clever solution is currently being used by ENS: whoever wants the handle the most would be willing to pay more for it, so auctioning the handle is a good way to go. The price starts at a high value and then decreases each second - giving everyone the chance to register it at the price they consider 'fair.' Linear pricing (e.g., the price decreases $5/hour) is unlikely to work because starting at a price no one would be willing to pay (e.g., $100 million) would result in an auction that is either too long or too short (the price decreases very rapidly). Instead, XCHandles implements an 'exponential premium' pricing strategy, where the auction starts with a very high value and the premium halves every 24 hours. The table below shows how the premium evolves over a 28-day auction when it starts at 100,000,000 wUSDC.b:&#x20;

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

More generally, auction puzzles will have the same return as a normal pricing puzzle, namely `(premium . registration_delta_seconds)`. The main difference is that the puzzle accepts three truths:

* `Buy_Time`: this is a verified timestamp that can be treated as 'time now.' The registry makes sure that the timestamp happened before the last transaction block's timestamp, but the user is financially motivated to provide a value as big (late) as possible since the auction puzzle would lower its premium for bigger timestamps.
* `Expiration`: the current expiration of the name, which was checked to be in the past. Note that expiration can't normally be 0.
* `Handle`: string containing the handle being auctioned off. Normally, the handle has already been validated by the other pricing puzzle.

The auction pricing puzzle may then accept an arbitrary solution. The exponential premium puzzle uses the order of truths to run the normal pricing in order to determine the 'true price' of a handle. The first two truths can be prepended to `pricing_program_solution`, which can then be used to evaluate the normal pricing puzzle (curried in via `BASE_PROGRAM` ). This pattern allows the exponential premium puzzle to add the time-dependent premium to a handle's base price, making sure the price at the end of auction is the same as the price if the handle were registered the first time (effectively 'ending' the auction).

_Written by_ [_yakuhito_](https://x.com/yakuh1t0) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Feb 15th, 2025._
