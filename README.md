---
description: Frequently Asked Questions about XCHandles
---

# XCHandles FAQ

### What is XCHandles?

XCHandles is, at its core, a decentralized address book. Users can register human-friendly handles such as @yak, which can then be associated with an XCH address, profile picture, display name, and much more. XCHandles is the Chia equivalent of the Ethereum Name Service and the first fully decentralized registry of this kind.

### How much does a handle cost?

New handles are priced based on length. Handles containing at least one number have a 50% discount. The current [pricing puzzle](techincal-manual/pricing-puzzles.md#factor-pricing) has a base price of 5.0 wUSDB.c, which translates to the costs below:

<table><thead><tr><th width="100">Length</th><th width="130" data-type="checkbox">Has Numbers?</th><th width="100">Factor</th><th width="173">Example Handle</th><th>Cost</th></tr></thead><tbody><tr><td>3</td><td>false</td><td>128</td><td>@abc</td><td>640 wUSDC.b per year</td></tr><tr><td>3</td><td>true</td><td>64</td><td>@ab1</td><td>320 wUSDC.b per year</td></tr><tr><td>4</td><td>false</td><td>64</td><td>@abcd</td><td>320 wUSDC.b per year</td></tr><tr><td>4</td><td>true</td><td>32</td><td>@abc1</td><td>160 wUSDC.b per year</td></tr><tr><td>5</td><td>false</td><td>16</td><td>@abcde</td><td>80 wUSDC.b per year</td></tr><tr><td>5</td><td>true</td><td>8</td><td>@a1234</td><td>40 wUSDC.b per year</td></tr><tr><td>6+</td><td>false</td><td>2</td><td>@example</td><td>10 wUSDC.b per year</td></tr><tr><td>6+</td><td>true</td><td>1</td><td>@example1</td><td>5 wUSDC.b per year</td></tr></tbody></table>

### What happens when a handle expires?

Immediately after a handle expires, an [expiration auction](techincal-manual/pricing-puzzles.md#exponential-premium) begins. The handle is valued at the normal price plus a premium. The premium begins at \~100.000.000 wUSDC.b and decreases continuously each second, halving each day. The first user to pay the current auction price will receive the handle. The premium becomes 0 28 days after an auction started, allowing anyone to register the handle at its normal price if no bids have been made.

### How should I refer to XCHandles?

When speaking, you can refer to XCHandles simply as "handles." If you think the underlying blockchain is not shared knowledge, you may also say "X-C-H handles"/"Chia handles." In writing (e.g., messages, UIs, etc.), please write "XCHandles". The only exception to the previous code is Rust code, where "Xchandles" may be used to conform to variable naming standards.

### Who controls pricing?

A price singleton has the ability to update the payment CAT for handles. The singleton also has control over the pricing and expired pricing puzzles, enabling it to change the pricing strategy of XCHandles if needed. The pricing singleton is currently a 7-of-11 multisig controlled by [warp.green validators](https://docs.warp.green/#who-are-the-validators).&#x20;

### How do I know that XCHandles was properly deployed? <a href="#how-do-i-know-catalog-was-properly-deployed" id="how-do-i-know-catalog-was-properly-deployed"></a>

Anyone can use [this CLI tool](https://github.com/Yakuhito/slot-machine/) to verify the validity of XCHandle's mainnet deployment procedure by running `cargo r xchandles verify-deployment`.
