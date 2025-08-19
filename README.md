
# RandStacks- Secure Random Number Generator (Clarity Smart Contract)

A verifiable and secure random number generator implemented in [Clarity](https://docs.stacks.co/docs/clarity/clarity-overview), designed for use on the Stacks blockchain. This contract utilizes multiple entropy sources and state management to generate pseudo-random values in a tamper-resistant and verifiable manner.

---

## 🧠 Overview

Blockchain environments, by design, lack inherent randomness. This smart contract provides a **deterministic yet unpredictable** method of generating random numbers using a combination of on-chain data and user input. It mitigates manipulation risks by incorporating diverse entropy sources and by limiting repeatable calls within the same block.

---

## 🔐 Key Features

* Combines multiple entropy sources:

  * Stacks block header hashes (recent and previous)
  * Burn block header hash
  * Transaction sender
  * Internal nonce and entropy accumulator
* Provides randomness in:

  * `[0, max]` range
  * `[min, max]` range
* State-safe: prevents same-block multiple calls to avoid manipulation
* Fully verifiable and deterministic from historical chain data

---

## 🛠 Functions

### `get-random (user-seed (buff 32)) (max-value uint) → (response uint)`

Returns a pseudo-random number in the range `[0, max-value]`.

#### Parameters:

* `user-seed`: 32-byte user-provided seed (adds user-supplied entropy)
* `max-value`: The upper bound of the range (inclusive)

#### Returns:

* `(ok uint)` — random number between 0 and `max-value` inclusive
* Error codes:

  * `ERR_ZERO_RANGE (err u101)` if `max-value` is 0
  * `ERR_SAME_BLOCK (err u102)` if called more than once in a block

---

### `get-random-in-range (user-seed (buff 32)) (min-value uint) (max-value uint) → (response uint)`

Returns a pseudo-random number in the range `[min-value, max-value]`.

#### Parameters:

* `user-seed`: 32-byte buffer seed provided by the user
* `min-value`: Minimum value of the random range
* `max-value`: Maximum value of the random range

#### Returns:

* `(ok uint)` — random number in the range `[min-value, max-value]`
* Error codes:

  * `ERR_INVALID_RANGE (err u100)` if `min-value > max-value`
  * Other errors as inherited from `get-random`

---

### `get-entropy-state → (buff 32)`

Read-only function that returns the current value of the contract’s internal entropy accumulator.

This value evolves over time as random values are requested and can be used for debugging, analysis, or verification of randomness.

---

## 🧩 Internals

### Entropy Sources:

* `header-hash` of the **current and previous Stacks blocks**
* `header-hash` of the **previous burn block**
* **Transaction sender’s address** (converted to buffer)
* **Nonce** (internal state that increments with each call)
* **Entropy accumulator** (updated on every call with `sha256` of prior state and new entropy)

### Internal Utilities:

* `uint-to-buff` – converts a uint to a 4-byte buffer
* `buff-to-byte` – extracts a single byte buffer from a uint
* `combine-entropy` – aggregates all entropy sources and hashes them
* `extract-uint-from-buff` – reads a 64-bit uint from a 32-byte buffer
* `get-byte-at` – safely extracts a byte at an index
* `buff-to-uint` – converts a byte buffer to uint using index lookup

---

## 🧪 Example Usage

### Call from frontend:

```clarity
(get-random 0x0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef u100)
```

Returns a random number between `0` and `100`.

```clarity
(get-random-in-range 0xdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeef u10 u50)
```

Returns a number in the range `[10, 50]`.

---

## ❌ Error Codes

| Code                | Value        | Description                         |
| ------------------- | ------------ | ----------------------------------- |
| `ERR_INVALID_RANGE` | `(err u100)` | `min-value` > `max-value`           |
| `ERR_ZERO_RANGE`    | `(err u101)` | `max-value` = 0                     |
| `ERR_SAME_BLOCK`    | `(err u102)` | Called more than once in same block |

---

## 🔒 Security Considerations

* **Block height gating** ensures that repeated calls within the same block are rejected to prevent manipulation by miners or validators.
* **User-provided seed** ensures external entropy that is not known by the contract.
* **Entropy accumulator** and **nonce** make sure each call mutates internal state in a non-reversible way.

> This contract provides **deterministic pseudo-randomness**. While secure within the assumptions of the blockchain environment, it's not suitable for high-stakes applications like cryptographic key generation or lottery jackpot disbursement without further external randomness (e.g., via an oracle).

---

## 🧱 Deployment Notes

Ensure your Clarity environment supports:

* `get-block-info?`
* `get-burn-block-info?`
* `to-consensus-buff?`
* Safe buffer operations (`element-at`, `index-of`, `concat`, `sha256`)

This contract is compatible with Stacks 2.1 and later.

---

## 📄 License

MIT License. Use freely with attribution. Contributions welcome.

--