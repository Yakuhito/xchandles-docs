---
description: Behold, the public API
---

# API

The listener API is a read-only JSON HTTP service used to resolve handles, look up data needed for registration, and get information about the protocol.

| Network | Base URL |
| ------- | -------- |
| Mainnet | [https://api.xchandles.com](https://api.xchandles.com) |

All listed routes are **GET** (HEAD is also supported), and CORS allows any origin. IDs in responses are 64-character lowercase hex **without** `0x`. Handles are 3-63 lowercase ASCII letters and digits with no normalization.

On most routes, omitting `launcher_id` selects the root registry, whose launcher is the default. If the index is too stale, important reads will return `503` with `"code": "index_stale"` instead of a normal response.

---

## `GET /healthz`

Liveness probe; returns an empty `200`.

---

## `GET /handle/{handle}`

Returns a live (unspent) slot plus information about the resolved singleton (if available, name NFT data as well) for a given handle. The data can be used to construct the coin id of the latest unspent handle slot and the latest NFTs, which can be checked on-chain to exist but not be spent for confirmation. `slot_parent_id` is the parent of that slot coin; `slot_parent_parent_id` and `slot_parent_inner_puzzle_hash` are its compact lineage proof (parent amount is the registry singleton amount, typically `1`). Together they are enough to spend the slot trustlessly. `pending_transfer` is `null` when no transfer is executable, or the same object as `GET /handle/{handle}/pending-transfer`.

| Param | Required | Description |
| ----- | -------- | ----------- |
| `{handle}` | yes | handle (in path) |
| `launcher_id` | no | Registry launcher id (64-char hex) |
| `include_metadata` | no | If `true`, include NFT metadata CLVM as hex (when available) |
| `bypass_expiration_safety_check` | no | If `true`, return the proof even after expiration (`410` otherwise) |

```json
{
  "registry_launcher_id": "<hex32>",
  "handle": "alice",
  "slot": {
    "counter": 0,
    "handle_hash": "<hex32>",
    "neighbors": { "left_value": "<hex32>", "right_value": "<hex32>" },
    "expiration": 4102444800,
    "owner_launcher_id": "<hex32>",
    "resolved_launcher_id": "<hex32>"
  },
  "slot_parent_id": "<hex32>",
  "slot_parent_parent_id": "<hex32>",
  "slot_parent_inner_puzzle_hash": "<hex32>",
  "slot_confirmation_height": 90,
  "resolved_singleton": { "...same shape as GET /singletons/{launcher_id}..." },
  "indexed_peak_height": 116,
  "pending_transfer": null
}
```

---

## `GET /handle/{handle}/pending-transfer`

Returns a performable pending ownership transfer for the handle, or `204` if none is executable. The same payload is also embedded as `pending_transfer` on `GET /handle/{handle}` (`null` instead of `204`). This should be queried and asserted to return 204 (or `pending_transfer: null`) when an offer involving a handle NFT is accepted.

| Param | Required | Description |
| ----- | -------- | ----------- |
| `{handle}` | yes | handle (in path) |
| `launcher_id` | no | Registry launcher id |

`200`:

```json
{
  "new_owner_launcher_id": "<hex32>",
  "new_resolved_launcher_id": "<hex32>",
  "update_confirmation_height": 100,
  "minimum_execution_height": 150,
  "initiator_coin_id": "<hex32>",
  "executor_coin_id": "<hex32>"
}
```

---

## `GET /singletons/{launcher_id}`

Returns information about a followed singleton. The API tracks all owner and resolved singletons for all handles registered in followed registries.

| Param | Required | Description |
| ----- | -------- | ----------- |
| `{launcher_id}` | yes | 64-char hex launcher id |
| `include_metadata` | no | If `true`, include NFT metadata CLVM as hex (when available) |

```json
{
  "launcher_id": "<hex32>",
  "parent_coin_id": "<hex32>",
  "amount": 1,
  "inner_puzzle_hash": "<hex32>",
  "confirmation_height": 100,
  "melted": false,
  "melt_height": null,
  "nft": {
    "metadata_treehash": "<hex32>",
    "metadata_updater_puzzle_hash": "<hex32>",
    "current_owner": null,
    "royalty_puzzle_hash": "<hex32>",
    "royalty_basis_points": 500,
    "p2_puzzle_hash": "<hex32>"
  },
  "indexed_peak_height": 116
}
```

`nft` is `null` for non-NFT singletons. `nft.metadata` is omitted unless `include_metadata=true`.

---

## `GET /registrations/{handle}`

Returns the latest confirmed register or expire action for a handle.

| Param | Required | Description |
| ----- | -------- | ----------- |
| `{handle}` | yes | handle (in path) |
| `launcher_id` | no | Registry launcher id |

```json
{
  "handle": "alice",
  "registration_secret": "<hex32>",
  "action_kind": "register",
  "protocol_fee": 1000,
  "confirmation_height": 90,
  "indexed_peak_height": 116
}
```

`action_kind` is `"register"` or `"expire"`. Fees are mojos of the current payment CAT.

---

## `GET /recent-registrations`

Returns the newest register/expire events, plus the running count of registered handles. This is used for the front page of XCHandles.

| Param | Required | Description |
| ----- | -------- | ----------- |
| `launcher_id` | no | Registry launcher id |
| `limit` | no | Page size; default and maximum `50` |

```json
{
  "items": [
    { "handle": "carol", "action_kind": "expire", "confirmation_height": 110 }
  ],
  "total_registered": 2,
  "indexed_peak_height": 116
}
```

---

## `GET /expiring`

Returns a cursor-paginated directory of handles that are in an expiration auction or about to expire.

| Param | Required | Description |
| ----- | -------- | ----------- |
| `view` | yes | `active` (in the 28-day auction) or `soon` (expires within 30 days) |
| `cursor` | no | Obtained from a previous page; used for pagination |
| `limit` | no | Page size; default and maximum `50` |
| `launcher_id` | no | Registry launcher id |

`view=active`:

```json
{
  "items": [{
    "handle": "bob",
    "expiration": 1798272000,
    "projected_pricing_timestamp": 1800000420,
    "current_premium": 94830,
    "total_registration_fee": 734830,
    "base_registration_fee": 640000,
    "reaches_base_at": 1800691200
  }],
  "next_cursor": "v1.1798272000.bob",
  "indexed_peak_height": 116,
  "confirmed_timestamp": 1800000000
}
```

`view=soon`:

```json
{
  "items": [{
    "handle": "dan",
    "expiration": 1800604800,
    "base_registration_fee": 640000
  }],
  "indexed_peak_height": 116,
  "confirmed_timestamp": 1800000000
}
```

`next_cursor` is omitted on the last page. Fees are mojos for a 1-year registration. Quotes use the **effective** base: the last generation whose `activation_timestamp` is at or before `confirmed_timestamp` (what a buyer pays after unrolling). The directory does not include the generation list or remaining unrolls.

---

## `GET /price`

Live Price Singleton snapshot: committed (pre-unroll) base, remaining executable unrolls, and index freshness. Optional `launcher_id` selects a registry; omitting it uses the default.

| Param | Required | Description |
| ----- | -------- | ----------- |
| `launcher_id` | no | Registry launcher id |

```json
{
  "indexed_peak_height": 116,
  "confirmed_timestamp": 1786935600,
  "current_base_price": 1,
  "unrolls": [
    { "activation_timestamp": 1786885200, "base_price": 9 },
    { "activation_timestamp": 1786892400, "base_price": 8 }
  ]
}
```

`current_base_price` is the committed registry / pricing puzzle (before unrolling). `unrolls` are remaining generations from the current scheduler generation onward, in order. A row with `activation_timestamp <= confirmed_timestamp` is **due** (a wallet can unroll now). The payable base after unrolling is the last due unroll’s `base_price`, or `current_base_price` if nothing is due. Before the first generation row that value is launch price `1`. Stale index: `503` `index_stale`.

---

## `GET /schedule`

The full static generation list baked into the Price Singleton launcher, including already-activated rows. This list does not change; the last row’s `base_price` holds after that timestamp. There are no peak or timestamp freshness fields. Optional `launcher_id` is the same as on other routes.

| Param | Required | Description |
| ----- | -------- | ----------- |
| `launcher_id` | no | Registry launcher id |

```json
{
  "generations": [
    { "activation_timestamp": 1786885200, "base_price": 9 },
    { "activation_timestamp": 1787022000, "base_price": 1 }
  ]
}
```

Clients use `/schedule` for future window math (reminder crossings, home tables). Do not treat a missing later row as a vault-era fallback.

---

## `GET /neighbors`

Given a handle hash, returns the neighboring handle slots, each slot's parent coin id, and its compact lineage proof (`parent_parent_id` and `parent_inner_puzzle_hash`; parent amount is always `1`). This endpoint alone (plus on-chain data) is enough to enable trustless registrations for light clients.

| Param | Required | Description |
| ----- | -------- | ----------- |
| `launcher_id` | yes | Registry launcher id |
| `handle_hash` | yes | handle tree hash (64-char hex) |

```json
{
  "left_handle_hash": "<hex32>",
  "right_handle_hash": "<hex32>",
  "left_left_handle_hash": "<hex32>",
  "left_expiration": 0,
  "left_counter": 0,
  "left_owner_launcher_id": "<hex32>",
  "left_resolved_launcher_id": "<hex32>",
  "left_parent_id": "<hex32>",
  "left_parent_parent_id": "<hex32>",
  "left_parent_inner_puzzle_hash": "<hex32>",
  "right_right_handle_hash": "<hex32>",
  "right_expiration": 0,
  "right_counter": 0,
  "right_owner_launcher_id": "<hex32>",
  "right_resolved_launcher_id": "<hex32>",
  "right_parent_id": "<hex32>",
  "right_parent_parent_id": "<hex32>",
  "right_parent_inner_puzzle_hash": "<hex32>"
}
```

---

## Errors

Chain-backed reads other than `/neighbors` use:

```json
{
  "code": "handle_not_found",
  "message": "No live handle slot is indexed for this handle",
  "request_id": "<uuid>"
}
```

| Status | `code` |
| ------ | ------ |
| `400` | `invalid_handle`, `invalid_launcher_id`, `invalid_view` |
| `404` | `handle_not_found`, `registry_not_followed`, `singleton_not_followed` |
| `410` | `handle_expired` |
| `503` | `index_stale`, `singleton_incomplete`, `singleton_mismatch`, `resolution_incomplete`, `resolution_mismatch` |
