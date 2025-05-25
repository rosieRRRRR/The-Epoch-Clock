# 100 Year Epoch Clock

This repository defines a **100-year decentralized UTC epoch clock** for Bitcoin-native smart contracts.

* **timer\_id**: `utc-epoch-100yr`
* **start\_timestamp**: 2025-05-25 00:00:00 UTC (Unix: `1748400000`)
* **duration\_seconds**: `3153600000` (100 years)
* **format**: `unix_seconds`
* **author**: Rosie

---

## ✅ Purpose

This clock acts as a **public, immutable UTC time anchor** for coordinating contract expiries across Bitcoin-native systems. It can be used in:

* Lending protocols
* Escrow contracts
* Auctions
* Scheduled token unlocks
* Any expiry-driven event on Bitcoin

It requires **no oracles**, **no server clocks**, and is entirely **verifiable on-chain**.

---

## 🕒 How to Use

Contracts referencing this clock should store metadata like:

```json
{
  "timer_id": "utc-epoch-100yr",
  "start_timestamp": <timestamp>,
  "duration_seconds": <duration>,
  "expiry_timestamp": <start + duration>
}
```

---

### 📍 Start Time Rule

Use the **block header timestamp** of the **confirmed broadcast transaction** as the `start_timestamp`.

* ✅ The time is derived from **on-chain data**
* ✅ It is **verifiable and immutable**
* ✅ It reflects when the **contract became active**
* ✅ It can be pulled directly from any Bitcoin block explorer or node

---

### 🧹 Example: Generic Contract

```json
{
  "timer_id": "utc-epoch-100yr",
  "start_timestamp": 1748500000,
  "duration_seconds": 604800,
  "expiry_timestamp": 1749104800
}
```

---

### 📊 Field Reference

| Field              | Description                                                                            |
| ------------------ | -------------------------------------------------------------------------------------- |
| `timer_id`         | Identifies the standard epoch clock (`utc-epoch-100yr`)                                |
| `start_timestamp`  | UTC time when the contract begins — taken from the block timestamp of the confirmed TX |
| `duration_seconds` | Duration the contract remains active (here: 7 days = 604800 seconds)                   |
| `expiry_timestamp` | Deterministic: `start_timestamp + duration_seconds`                                    |

---

## 🧠 Use Cases

This structure is suitable for:

* 🔐 **Time-locked vaults** — prevent early withdrawals
* 🗳️ **Voting windows** — define open and close times
* 🎯 **Claim windows** — limit reward access to a fixed period
* 🛍️ **Timed auctions** — define clear start and expiry

---

## 💻 Example Logic (Client-Side)

```js
const now = Math.floor(Date.now() / 1000); // Current UTC time
if (now < start_timestamp) {
  show("Not yet active");
} else if (now < expiry_timestamp) {
  showCountdown(expiry_timestamp - now);
  allowRepayment();
} else {
  show("Expired");
  enableLiquidation();
}
```

---

## 🔗 Inscription

* **Inscription ID**: `96733659`
* **View on ordinals.com**:
  [https://ordinals.com/inscription/96733659](https://ordinals.com/inscription/96733659)

---

## 🧠 Designed by Rosie

Created to support **accurate, trustless time-based coordination** across Bitcoin — from DeFi lending to programmable unlocks.

> No oracles. No drift. Just time, anchored to chain.
