# LumenFlow Contract API Reference

This document describes all public read and write functions exposed by the LumenFlow smart contract, with an emphasis on the data structures returned.

---

## `get_contract_config`

Returns all current admin-configurable parameters as a single `ContractConfig` struct.

### Signature

```rust
pub fn get_contract_config(env: Env) -> ContractConfig
```

### Access

Read-only. No authentication required. Callable by anyone.

### Returns — `ContractConfig`

| Field | Type | Default | Description |
|---|---|---|---|
| `platform_fee_bps` | `u32` | `0` | Platform fee in basis points (100 bps = 1%). Set via `set_platform_fee`. |
| `refund_window_secs` | `u64` | `2_592_000` | Refund eligibility window in seconds (default 30 days). Set via `set_refund_window`. |
| `min_refund_amount` | `i128` | `1` | Minimum allowed refund amount in token base units. Set via `set_min_refund_amount`. |
| `multisig_expiry_duration_secs` | `u64` | `604_800` | Duration in seconds before an unexecuted multisig payment is considered expired (default 7 days). Set via `set_multisig_expiry_duration`. |
| `max_refunds_per_order` | `u32` | `5` | Maximum number of pending refund requests allowed per payment order. Set via `set_max_refunds_per_order`. |
| `payment_cleanup_period_secs` | `u64` | `2_592_000` | Payment record age (in seconds) before eligibility for automatic cleanup (default 30 days). Set via `set_payment_cleanup_period`. |
| `large_payment_threshold` | `i128` | `10_000_000` | Payment amounts at or above this threshold trigger a `suspicious_activity` event. Set via `set_large_payment_threshold`. |

### Example (Stellar CLI)

```bash
stellar contract invoke \
  --id $CONTRACT_ID \
  --source-account $CALLER_KEY \
  --network $NETWORK \
  -- get_contract_config
```

### Example (JavaScript / stellar-sdk)

```javascript
const config = await contract.getContractConfig();
console.log(`Platform fee: ${config.platform_fee_bps} bps`);
console.log(`Refund window: ${config.refund_window_secs / 86400} days`);
console.log(`Min refund amount: ${config.min_refund_amount}`);
```

---

## Admin Configuration Functions

The following admin-only functions update parameters returned by `get_contract_config`. Each emits a `lumenflow/config_updated` event with the old and new values. See [events-reference.md](events-reference.md) for event payload details.

| Function | Parameter Changed | Event Emitted |
|---|---|---|
| `set_platform_fee(admin, fee_bps)` | `platform_fee_bps` | `config_updated` |
| `set_refund_window(admin, window_secs)` | `refund_window_secs` | `config_updated` |
| `set_min_refund_amount(admin, min_amount)` | `min_refund_amount` | `config_updated` |
| `set_multisig_expiry_duration(admin, duration_secs)` | `multisig_expiry_duration_secs` | `config_updated` |
| `set_max_refunds_per_order(admin, max)` | `max_refunds_per_order` | _(none)_ |
| `set_payment_cleanup_period(admin, period)` | `payment_cleanup_period_secs` | _(none)_ |
| `set_large_payment_threshold(admin, threshold)` | `large_payment_threshold` | _(none)_ |

---

## `get_global_payment_stats`

Returns aggregate statistics. Admin only.

### Signature

```rust
pub fn get_global_payment_stats(
    env: Env,
    admin: Address,
    date_start: Option<u64>,
    date_end: Option<u64>,
) -> Result<GlobalStats, PaymentError>
```

### Returns — `GlobalStats`

| Field | Type | Description |
|---|---|---|
| `total_payments` | `u32` | Total number of completed payments. |
| `total_volume` | `i128` | Aggregate completed payment volume (saturating sum). |
| `total_refunds` | `u32` | Total number of executed refunds. |
| `total_refund_volume` | `i128` | Aggregate refund volume (saturating sum). |
| `active_merchants` | `u32` | Number of currently active (non-deactivated) merchants. |

---

## `get_merchant`

Returns the full merchant profile for a registered address.

### Signature

```rust
pub fn get_merchant(env: Env, merchant_address: Address) -> Result<Merchant, PaymentError>
```

### Returns — `Merchant`

| Field | Type | Description |
|---|---|---|
| `address` | `Address` | The merchant's Stellar address. |
| `name` | `String` | Business name. |
| `description` | `String` | Business description. |
| `contact_info` | `String` | Contact information (email, website, etc.). |
| `category` | `MerchantCategory` | `Retail` \| `Food` \| `Services` \| `Digital` \| `Other` |
| `active` | `bool` | Whether the merchant can currently accept payments. |
| `verified` | `bool` | Admin-granted verification badge. |
| `registered_at` | `u64` | Ledger timestamp of registration. |
| `total_received` | `i128` | Cumulative amount received across all completed payments. |

---

*For the full event reference, see [events-reference.md](events-reference.md).*  
*For the refund lifecycle, see [refund-lifecycle.md](refund-lifecycle.md).*  
*For auth model details, see [auth-model.md](auth-model.md).*
