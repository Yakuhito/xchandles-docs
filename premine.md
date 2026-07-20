# Premine

## Premine

The premine is the set of handles reserved before open mainnet registration and assigned to specific receivers. It exists so people who already held names in earlier Chia naming systems - and contributors who helped the ecosystem - receive a fair starting allocation, while everyone else still has a fair shot at registering remaining handles when XCHandles launches.

Mainnet launch is scheduled for **August 20, 2026 at 09:00 UTC**. Alongside the premine, the first two weeks after launch use a declining global pricing factor so early public registrations cost more than the long-term price. That launch pricing period is described in [CHIP-0054](https://github.com/Chia-Network/chips/pull/192): handle prices are multiplied by a factor that steps down each day and eventually reaches **1x** (normal price). The factor schedule is:

| Time   | Pricing Factor |
| ------ | -------------- |
| Day 1  | 1000x          |
| Day 2  | 750x           |
| Day 3  | 500x           |
| Day 4  | 250x           |
| Day 5  | 100x           |
| Day 6  | 75x            |
| Day 7  | 50x            |
| Day 8  | 33x            |
| Day 9  | 25x            |
| Day 10 | 15x            |
| Day 11 | 10x            |
| Day 12 | 5x             |
| Day 13 | 3x             |
| Day 14 | 2x             |

After day 14, the factor settles at **1x**. Normal length-based pricing (including the discount for handles that contain numbers) still applies underneath the global factor - see How much does a handle cost?.

Together, the premine and the launch pricing period aim at the same outcome: desirable handles should go to people who want them most, rather than whoever can script the fastest claim at a flat low price.

### Two sources

The published premine has two parts:

1. **Base premine (CNS + NamesDAO)** - handles derived from viable [Chia Name Service](https://mintgarden.io/collections/chia-name-service-col10r992w4cvasaxjs7ldc0n5hlhl5dklc3x3l2tp405ra6adzczqksnw49f2) and [NamesDAO](https://mintgarden.io/collections/.xch-namesdao-names-col1u9pemm2avjcz8t9emhga4vys5knugsfnctpkk2jyx05jc8d6ch2swe4qvm) legacy registrations as of the **migration cutoff** (**July 20, 2026 at 09:00 UTC**). After that instant, ownership and registration changes on those systems no longer affect the base premine.
2. **Contribution premine** - handles reserved for Chia ecosystem contributors, collected separately and merged into the same published file before launch.

You can look up whether a handle is reserved on the XCHandles launch site. The canonical published file is [`premine.csv`](https://github.com/Yakuhito/nfts/blob/master/premine.csv).

### Base premine (CNS and NamesDAO)

At a high level:

* A legacy name can become a handle if, after removing the `.xch` substring (and everything that comes after it), it is already a valid XCHandles handle (**3-63** characters, `a-z` and `0-9` only), or it becomes valid after stripping `-` and `_`.
* CNS is resolved first. NamesDAO only fills handles that CNS did not claim.
* When several records compete for the same handle, deterministic rules pick a single winner (exact spellings beat stripped ones; active registrations beat expired ones; earlier mint time wins remaining ties).
* The premine recipient is the NFT’s owner address at the migration cutoff - specifically the inner puzzle hash encoded as an XCH address - not a forwarding address from metadata.
* Each base-premine handle expires at `max(legacy expiration, launch instant) + 122 days`, so holders keep a meaningful registration window after launch even if their legacy name was near expiry.

Allocation type in the CSV is `cns` or `namesdao`. The allocation explanation links to the winning NFT’s MintGarden page.

### Contribution premine

Handles reserved for ecosystem contributors appear in the published premine with allocation type `contributor`. All contributor premine entries are found in [`contributor-premine.csv`](https://github.com/Yakuhito/nfts/blob/master/contributor-premine.csv)

Contribution requests are messages to [@yakuhito](https://x.com/yakuhito) on X that include a receive address, the requested handle or handles, and a short description of contributions. The contribution deadline is **August 3, 2026 at 09:00 UTC** - requests after that instant are not considered. Confirmed allocations are merged into [`premine.csv`](https://github.com/Yakuhito/nfts/blob/master/premine.csv) before launch. Contribution-premine handles expire on **August 20, 2027 at 09:00 UTC** (one year after launch).

If a handle that is already present in the base premine is requested, the recipient will NOT be changed, but the expiration will be updated to **August 20, 2027 at 09:00 UTC** (one year after launch). All handles updated as such can be found in [`contribuor-extensions.csv`](https://github.com/Yakuhito/nfts/blob/master/contributor-extensions.csv).

### Full selection details

This section is for readers who want the precise base-premine rules. Contribution allocations remain a manual review process; they are not produced by the algorithm below.

#### Eligibility

1. Start from CNS NFTs under creator address `xch1zdfcemh4cvcglzx03qu0czlaurt800agghz86c0m5uez4p30dvls8zjc8l` and NamesDAO NFTs under DID `did:chia:13myvry7hmp6nwpa00lqexczka652xkyujyjsecplge8c65rtdl4qd0yya7`.
2. Read the legacy name from off-chain metadata (hash-verified). Remove the `.xch` substring if it exists, as well as anything that comes after it.
3. Classify the result:
   * **Exact** - already a valid handle (`a-z` / `0-9`, length 3-63).
   * **Stripped** - contains `-` or `_`, and removing those characters yields a valid handle.
   * Otherwise the record is skipped (recorded in the warnings CSV when generating from chain data).
4. Derive **legacy expiration** from metadata (not from whether the NFT coin is spent):
   * **CNS** - calendar date `YYYY-MM-DD` interpreted as **23:59:59 UTC** that day.
   * **NamesDAO** - `Expiry` block height projected to UNIX time with a fixed formula: heights below `9_000_000` map to expiration `0`; otherwise `ceil((height - 9_000_001) × 18.75) + 1_783_933_174` (the UNIX time of block `9_000_001` is `1_783_933_174`).
5. A registration is **active** at the migration cutoff if its legacy expiration is strictly after `2026-07-20 09:00:00 UTC`; otherwise it is **expired**. Both active and expired records can still win under the rules below.

**Registration time** is the timestamp of the block in which the NFT’s launcher coin was spent (eve mint), not a MintGarden listing date.

#### Per-source resolution (CNS, then NamesDAO)

For each source independently:

1. **Succession** - group records that share the same original legacy string. Prefer the latest **active** registration time; if none are active, prefer the latest expired registration time. Identical timestamps break ties toward the lexicographically smallest NFT id.
2. **Collisions** - group the surviving records by the derived handle. Prefer **Exact** over **Stripped**, then active over expired, then earliest registration time, then lexicographically smallest NFT id.
3. After CNS is fully resolved, run the same process for NamesDAO, but **skip any handle already claimed by CNS**.

Burned NFTs are still eligible: the recipient is whatever inner puzzle hash the NFT coin had at the migration cutoff (including a burn puzzle hash), encoded as an XCH address.

#### Expiration written at launch

* Base premine: `max(legacy expiration, 2026-08-20 09:00:00 UTC) + 122 days`.
* Contribution premine: `2027-08-20 09:00:00 UTC`.

#### Published CSV columns

```
handle,recipient,expiration,allocation_type,allocation_explanation
```

* `allocation_type` - `cns`, `namesdao`, or `contributor`
* `allocation_explanation` - for base premine, the MintGarden NFT URL for the winning registration (`https://mintgarden.io/nfts/{nft_id}`); for contributors, a URL documenting the allocation
* `expiration` - integer UNIX timestamp (UTC)

Generation from a local chain snapshot also writes a companion warnings CSV (`reason,source,nft_id,original_name,metadata_urls`) for records that could not be turned into candidates (for example missing name or expiration metadata).

### Verifying the CNS / NamesDAO premine

Anyone can independently rebuild and check the **base** premine (CNS + NamesDAO). Contribution rows are maintained separately and merged into the published file; do not expect a base-only rebuild to match [`premine.csv`](https://github.com/Yakuhito/nfts/blob/master/premine.csv) row-for-row unless you filter to `cns` and `namesdao` only.

The open-source tooling lives in the [`nfts`](https://github.com/Yakuhito/nfts) repository. After the migration cutoff, a trustworthy run looks like:

```bash
rm -f nfts.db nfts.db-shm nfts.db-wal

cargo run -- --db nfts.db add \
  'xch1zdfcemh4cvcglzx03qu0czlaurt800agghz86c0m5uez4p30dvls8zjc8l,did:chia:13myvry7hmp6nwpa00lqexczka652xkyujyjsecplge8c65rtdl4qd0yya7'

cargo run -- --db nfts.db sync

cargo run -- --db nfts.db premine generate \
  --output base-premine.csv \
  --warnings premine-warnings.csv

cargo run -- premine confirm base-premine.csv
```

`premine generate` rebuilds the base premine from a fresh local snapshot. `premine confirm` reconstructs the expected set from MintGarden collection and event APIs and compares it exhaustively to the CSV - it never modifies the input file, and exits nonzero on any mismatch.

Compare your confirmed `cns` / `namesdao` rows to the published [`premine.csv`](https://github.com/Yakuhito/nfts/blob/master/premine.csv). After launch, you can also use the [slot-machine](https://github.com/Yakuhito/slot-machine/) `xchandles verify-deployment` command to check that on-chain premine registrations match the trusted CSV bundled with that deployment - see How do I know that XCHandles was properly deployed?.

_Written by_ [_yakuhito_](https://x.com/yakuhito) _from_ [_FireAcademy.io_](https://fireacademy.io/) _on Jul 19th, 2026._
