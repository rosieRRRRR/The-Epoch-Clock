# Epoch Clock for Bitcoin

**A deterministic, decentralized time standard for on-chain contract enforcement**

---

## Clock Parameters

- **timer_id**: `utc-epoch-100yr`
- **start_timestamp**: 2025-05-25 00:00:00 UTC (Unix: `1748400000`)
- **duration_seconds**: `3153600000` (100 years)
- **format**: `unix_seconds`
- **author**: Rosie

---

## Overview

The 100 Year Epoch Clock introduces a shared, immutable concept of time for Bitcoin-native contracts.

By defining a fixed start point and deterministic duration logic, it enables Bitcoin scripts, PSBTs, and clients to coordinate contract expiry, vesting, unlocks, and enforcement - all without oracles, servers, or off-chain dependencies.

The result is reliable time-based logic on Bitcoin, enforced purely by the chain.

---

## How It Works

The Epoch Clock begins automatically when the defining transaction is confirmed in a block. Its start_timestamp is set to the block header timestamp (nTime) of that block.
This defining transaction could be a canonical anchor, PSBT, contract deployment, or inscription - any on-chain event used to trigger time-based logic.

No user interaction or activation is required - the chain itself provides the start signal.

This `start_timestamp` is a known UTC value, directly accessible via RPC or public block explorers. All expiry logic is calculated as:

```text
expiry_timestamp = start_timestamp + duration_seconds
```

Bitcoin Script uses this value to enforce expiry using standard primitives like `OP_CHECKLOCKTIMEVERIFY` and `nLockTime`.

---

> **Note for implementers**:  
> `nLockTime` values below 500,000,000 are treated as block heights, not timestamps.  
> To avoid ambiguity, always ensure expiry timestamps are above this threshold.  
>  
> Also note: block timestamps (`nTime`) are miner-controlled but bounded by Bitcoin consensus to be no earlier than the median of the last 11 blocks and no more than 2 hours into the future.  
> This ensures bounded determinism and prevents arbitrary manipulation, enabling predictable enforcement.

```text
  nLockTime = expiry_timestamp
  nSequence = 0xffffffff
```

---

## Client Logic Example (JavaScript)

```js
const start_timestamp = 1748500000;
const duration_seconds = 604800; // 7 days
const expiry_timestamp = start_timestamp + duration_seconds;
const now = Math.floor(Date.now() / 1000);

if (now < start_timestamp) {
  show("Not yet active");
} else if (now < expiry_timestamp) {
  showCountdown(expiry_timestamp - now);
  enableRepayment();
} else {
  show("Expired");
  allowLiquidation();
}
```

> Countdowns can be automated in any client by comparing `expiry_timestamp` against `Math.floor(Date.now() / 1000)` (Unix seconds in UTC). The display should always use UTC to align with on-chain timestamps.

---

## Reference Implementation

```json
{
  "timer_id": "utc-epoch-100yr",
  "start_timestamp": 1748400000,
  "duration_seconds": 604800,
  "expiry_timestamp": 1749004800
}
```

### Field Reference

| Field              | Description                                                          |
| ------------------ | -------------------------------------------------------------------- |
| `timer_id`         | Identifies the epoch clock standard                                  |
| `start_timestamp`  | UTC time when the clock begins (from block timestamp of defining TX) |
| `duration_seconds` | Contract-specific duration (e.g. 7 days = 604800 seconds)            |
| `expiry_timestamp` | Derived: `start_timestamp + duration_seconds`                        |

---

## Use Cases

Each of the following is enforceable on Bitcoin Layer 1 today using standard Script and PSBT features:

| Use Case                    | Description                                                    |
| --------------------------- | -------------------------------------------------------------- |
| Lending with Expiry         | Borrower repays before expiry; lender claims funds after       |
| Time-Locked Offers          | Offers automatically expire if not accepted                    |
| Auction Finalisation        | Winner finalises claim after auction closes                    |
| Rental Agreements           | Asset access expires and reverts to owner                      |
| Escrow with Delivery Window | Seller receives funds if delivery confirmed within time window |
| Vault Timelocks             | Funds unlock after fixed duration                              |
| Dead Man Switch             | If user inactive, fallback party can claim after expiry        |
| Staking Unlocks             | Stake becomes withdrawable after defined time                  |
| Multi-Epoch Vesting         | Tranches unlock at predefined intervals                        |

Implemented using:

- `OP_CHECKLOCKTIMEVERIFY`
- `OP_CHECKSEQUENCEVERIFY`
- `nLockTime` and `nSequence`
- PSBT coordination with optional multisig

---

## Developer Tools and Integrations

### Test Vectors

```json
[
  {
    "test_case": "1 hour contract",
    "start_timestamp": 1748400000,
    "duration_seconds": 3600,
    "expected_expiry_timestamp": 1748403600
  },
  {
    "test_case": "7 day contract",
    "start_timestamp": 1748400000,
    "duration_seconds": 604800,
    "expected_expiry_timestamp": 1749004800
  },
  {
    "test_case": "3 month contract",
    "start_timestamp": 1748400000,
    "duration_seconds": 7862400,
    "expected_expiry_timestamp": 1756262400
  },
  {
    "test_case": "5 year contract",
    "start_timestamp": 1748400000,
    "duration_seconds": 157680000,
    "expected_expiry_timestamp": 1906080000
  }
]
```

### JSON Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Epoch Clock Metadata",
  "type": "object",
  "properties": {
    "timer_id": {
      "type": "string",
      "description": "Identifies the epoch clock"
    },
    "start_timestamp": {
      "type": "integer",
      "description": "Unix timestamp from block.nTime"
    },
    "duration_seconds": {
      "type": "integer",
      "description": "Length of contract duration in seconds"
    },
    "expiry_timestamp": {
      "type": "integer",
      "description": "Computed expiry timestamp"
    }
  },
  "required": ["timer_id", "start_timestamp", "duration_seconds", "expiry_timestamp"]
}
```

### Miniscript Representation Example

```text
or_b(
  and_v(before(expiry_timestamp), pk(borrower_key)),
  pk(lender_key))
```

This means:

- If the borrower acts before expiry, they can repay
- Otherwise, lender can claim the funds afterward

---

### Developer Integration Checklist

1. **Resolve the inscription ID** (`96733659`) using a block explorer or indexer.
2. **Fetch the transaction's block** and extract its `nTime` (block timestamp).
3. **Use that `nTime`** as the `start_timestamp` for all contract logic.
4. **Calculate expiry** as `start_timestamp + duration_seconds`.
5. **Use standard Bitcoin Script** (`nLockTime`, `OP_CHECKLOCKTIMEVERIFY`) to enforce the expiry.
6. **Integrate metadata** in JSON using the defined schema for PSBT workflows, clients, or display.

---

## Metadata Format

```json
{
  "timer_id": "utc-epoch-100yr",
  "start_timestamp": 1748500000,
  "duration_seconds": 604800,
  "expiry_timestamp": 1749104800
}
```

---

## Inscription

**Inscription ID**: `96733659`  
**View**: [https://ordinals.com/inscription/96733659](https://ordinals.com/inscription/96733659)

To ensure global visibility and permanence, `utc-epoch-100yr` is anchored via Inscription `96733659`, which defines the clock and anchors its start timestamp to the Bitcoin blockchain. While any immutable transaction could serve as an anchor, this inscription enables trustless client resolution without external coordination.

---

## Rationale and Benefits

Bitcoin already supports time-based enforcement using `nLockTime`, `OP_CHECKLOCKTIMEVERIFY`, and `OP_CHECKSEQUENCEVERIFY`.

The Epoch Clock builds on these primitives by introducing:

- A **shared, named time anchor** (`timer_id`) usable across wallets, protocols, and contracts
- A **deterministic start point**, derived from the `nTime` of a specific, immutable Bitcoin block
- A **standardised metadata format** for expiry logic that can be displayed, verified, and reused
- A coordination layer enabling **multi-protocol composability**, especially for trustless DeFi and vault flows

While `nLockTime` and `OP_CLTV` are widely used, their implementations vary. The Epoch Clock provides a shared coordination standard.

---

## Trust Assumptions and Determinism

- The `start_timestamp` is defined by the **Unix timestamp (`nTime`)** in the block that confirms a specific, pre-agreed transaction (such as an inscription).
- Clients referencing the same `timer_id` must validate the source - e.g. by resolving the Inscription ID `96733659` or a known TXID - to derive the correct start time.
- Each `timer_id` must map to exactly one immutable transaction. This prevents malicious redefinition or `timer_id` collisions and ensures all systems reference a single, deterministic clock origin.

To be deterministic and trustless:

- Clients must hardcode or agree on the defining transaction (not just the name)
- All logic derived from the Epoch Clock must treat the `nTime` of that confirming block as the canonical start
- No reliance on external clocks or oracle feeds is required

---

## Vaults and Covenant Applications

The Epoch Clock can serve as a core coordination mechanism for covenant-based systems and vault architectures on Bitcoin.

Because its logic is deterministic and script-compatible, it integrates naturally with:

### Vault Trees (Pre-Signed or Script Trees)

```text
IF
  <expiry_timestamp> OP_CHECKLOCKTIMEVERIFY OP_DROP
  <fallback_path>
ELSE
  <normal_spend_path>
ENDIF
```

### Taproot Script Trees

Anchoring individual script leaves with time constraints:

- Inheritance contracts
- Delegated recovery
- Time-based key rotations

### Pre-Signed Covenant Protocols (e.g. Ark, BitVM Vaults)

Use epoch-based logic in pre-signed spend trees:

- Refunds valid after expiry
- Fallback or inactivity paths

---

## Author

Designed by Rosie, 2025  
This is not a consensus-layer protocol change. It is a coordination layer - opt-in, immutable, and enforced by the chain.


