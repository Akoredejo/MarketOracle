# MarketOracle: Decentralized ML-Driven Data Feed

**MarketOracle** is a high-integrity, decentralized oracle infrastructure engineered for the Stacks blockchain. I have designed this system to bridge the gap between sophisticated off-chain **Machine Learning (ML) models** and on-chain decentralized finance (DeFi). By utilizing a weighted consensus mechanism, cryptographic whitelisting, and economic "skin-in-the-game" via staking, MarketOracle ensures that market data is both resilient to manipulation and mathematically robust.

---

## Table of Contents

* Introduction and Vision
* Core Architectural Philosophy
* Systematic Workflow
* Detailed Function Reference: Private
* Detailed Function Reference: Public
* Detailed Function Reference: Read-Only
* Error Code Taxonomy
* Economic Security (Staking & Slashing)
* Administrative Governance
* Contribution Guidelines
* Full MIT License

---

## Introduction and Vision

I architected MarketOracle to solve the "Garbage In, Garbage Out" problem prevalent in many blockchain data feeds. In a world where ML models provide superior predictive capabilities but remain centralized, MarketOracle provides the decentralized "courtroom" where these models must prove their accuracy.

The system operates on the principle of **Robust Aggregation**. Instead of trusting a single data source, I have implemented a multi-signature style submission process where a quorum of independent ML agents must agree on a price before it is ever committed to the global state.

---

## Core Architectural Philosophy

The contract is built on three pillars:

1. **Identity & Reputation**: Only vetted models can participate, but their influence grows or shrinks based on historical performance.
2. **Economic Alignment**: The staking mechanism ensures that any attempt to "poison" the data pool results in immediate financial loss for the attacker.
3. **Efficiency**: By utilizing Clarity’s specialized data maps, I have optimized the contract to handle multi-asset data streams without unbounded gas growth.

---

## Systematic Workflow

The lifecycle of a single price point within the system follows this strict pipeline:

* **Whitelisting**: The `contract-owner` authorizes a principal representing an ML model.
* **Collateralization**: The oracle stakes `minimum-stake` (u1000) to activate their reporting status.
* **Submission**: Oracles push `uint` price predictions for a specific `string-ascii` asset.
* **Quorum Check**: Once the `consensus-threshold` is met, the `calculate-and-update-consensus` function is triggered.
* **Verification**: The system calculates the mean, updates the public ledger, and clears the buffer.

---

## Detailed Function Reference: Private

These internal methods form the "brain" of the contract. I have kept them private to ensure that state changes like reputation adjustment and slashing can only be triggered by the contract's internal consensus logic.

### `is-oracle`

* **Input**: `(user principal)`
* **Logic**: Performs a `map-get?` on the reputation ledger.
* **Purpose**: Validates if a user has been whitelisted.

### `check-active`

* **Input**: None
* **Logic**: Asserts the `is-paused` variable is `false`.
* **Purpose**: Acts as a global circuit breaker check for all state-changing operations.

### `calculate-average`

* **Input**: `(values (list 10 uint))`
* **Logic**: Uses a functional `fold` to sum the list and divides by `(len values)`.
* **Purpose**: Provides the core mathematical mean for price consensus.

### `slash-oracle`

* **Input**: `(oracle principal)`
* **Logic**: Deducts `slash-amount` from `oracle-stakes` and resets `oracle-reputation` to `u0`.
* **Purpose**: Hard-coded punishment for malicious or extreme-outlier behavior.

### `reward-oracle`

* **Input**: `(oracle principal)`
* **Logic**: Increments the reputation counter by `u1`.
* **Purpose**: Incentivizes consistent, honest participation in the network.

---

## Detailed Function Reference: Public

Public functions are the primary interface for oracles and the contract administrator.

### `add-oracle`

* **Access**: Owner Only.
* **Description**: Initializes a new principal into the system with a starting reputation of `u100`.

### `stake-tokens`

* **Access**: Any authorized oracle.
* **Description**: Deposits tokens into the contract’s internal accounting. I have set a `minimum-stake` of `u1000` to ensure significant commitment.

### `submit-prediction`

* **Access**: Authorized Oracles with sufficient stake.
* **Description**: Appends a price to the `pending-submissions` map. I included an `index-of` check to prevent the same oracle from voting twice in a single round.

### `calculate-and-update-consensus`

* **Access**: Public (Incentivized trigger).
* **Description**: The most computationally intensive function. It validates the threshold, calculates the average, updates the `verified-prices` ledger, rewards the participants, and flushes the pending state for that asset.

### `report-outlier`

* **Access**: Public.
* **Description**: Allows the community to police the oracle. If a submitted price deviates by >20% from the eventual consensus, this function triggers the `slash-oracle` routine.

---

## Detailed Function Reference: Read-Only

These functions provide gas-less access to the oracle's data for other smart contracts or front-end applications.

### `get-verified-price`

* **Input**: `(asset (string-ascii 32))`
* **Returns**: The last finalized price for the given asset.

### `get-last-consensus-block`

* **Input**: `(asset (string-ascii 32))`
* **Returns**: The block height of the last update. Essential for checking data "freshness."

### `get-oracle-reputation`

* **Input**: `(oracle principal)`
* **Returns**: The current reputation score of a specific node.

### `is-price-valid`

* **Input**: `(asset, price-check, tolerance)`
* **Returns**: A boolean indicating if a provided price is within a specific range of the oracle's price. I designed this for third-party protocols to use as a secondary verification step.

---

## Error Code Taxonomy

| Error Code | Name | Logic Trigger |
| --- | --- | --- |
| `u100` | `err-owner-only` | Unauthorized admin attempt. |
| `u101` | `err-not-authorized` | Principal is not in the oracle whitelist. |
| `u102` | `err-already-submitted` | Duplicate entry in the pending list. |
| `u103` | `err-consensus-not-reached` | Submission count < `consensus-threshold`. |
| `u104` | `err-no-data` | Querying an asset with no history. |
| `u105` | `err-insufficient-stake` | Oracle balance < `minimum-stake`. |
| `u106` | `err-contract-paused` | Operation blocked by circuit breaker. |
| `u107` | `err-invalid-amount` | Stake/Withdrawal amount is illogical. |
| `u109` | `err-within-tolerance` | A report was filed against an honest oracle. |

---

## Economic Security

### Staking & Slashing

To prevent "lazy" or "malicious" reporting, I have implemented a slashable collateral system. If an ML model is compromised and begins reporting prices that deviate significantly (20%+) from the aggregate mean of its peers, it faces a penalty of `u500` and a total loss of reputation. This makes the cost of an attack , where  is the potential gain from market manipulation.

---

## Administrative Governance

The `contract-owner` retains specific powers to ensure the system's longevity:

1. **Circuit Breaker**: `set-paused` can halt all activity during extreme market volatility.
2. **Threshold Tuning**: `update-threshold` allows the admin to increase the required number of oracles as the network grows.
3. **Emergency Recovery**: `emergency-withdraw` ensures that if the contract is paused due to a bug, funds can be recovered and migrated.

---

## Contribution Guidelines

I welcome the developer community to audit and enhance this oracle. Areas of interest include:

* Implementing **Median** calculation instead of Simple Average to further increase outlier resistance.
* Adding **Weighted Averages** where the `reputation` score acts as a multiplier for an oracle's vote.
* Integrating with the **Stacks Token (STX)** for actual asset transfers during staking.

---

## License

```text
MIT License

Copyright (c) 2026 MarketOracle Project

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

```
