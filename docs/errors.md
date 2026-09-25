# LumenFlow Error Reference

All errors returned by the LumenFlow contract are typed values of the `PaymentError` enum, serialized as `u32` error codes in the Soroban XDR envelope.

---

## Error Codes

### Auth Errors

| Code | Name | Description |
|------|------|-------------|
| `1` | `Unauthorized` | The caller does not have permission to perform this action. |
| `2` | `AdminAlreadySet` | The contract admin has already been initialized via `set_admin`. |
| `3` | `InvalidAdminAddress` | The provided admin address is a contract address, which is not permitted. |

---

### Merchant Errors

| Code | Name | Description |
|------|------|-------------|
| `10` | `MerchantNotFound` | No merchant profile exists for the given address. |
| `11` | `MerchantAlreadyRegistered` | A merchant profile already exists for this address. |
| `12` | `MerchantInactive` | The merchant has been deactivated and cannot accept payments. |

---

### Payment Errors

| Code | Name | Description |
|------|------|-------------|
| `20` | `PaymentNotFound` | No payment record exists for the given order ID. |
| `21` | `PaymentAlreadyExists` | A payment with this order ID has already been processed. |
| `22` | `InvalidAmount` | The payment amount must be strictly greater than zero. |
| `23` | `InvalidSignature` | The provided ed25519 signature is invalid or does not match the public key. |
| `24` | `PaymentExpired` | The payment request TTL has elapsed; the request has expired. |
| `25` | `InsufficientBalance` | The payer's token balance is insufficient for this payment. |
| `26` | `TokenNotAllowed` | The token address is not on the contract's allowed-token list. See note below. |

> **`TokenNotAllowed` (code 26):** Raised by `process_payment_with_signature`, `batch_payment`, `initiate_multisig_payment`, and **`create_payment_request`** when the supplied `token_address` has not been whitelisted by the admin via `add_allowed_token`. This check happens **at creation time** for payment requests, preventing a confusing failure at pay time.
>
> **Resolution:** Ask the contract admin to call `add_allowed_token(admin, token_address)` to whitelist the token before creating payment requests or processing payments with it.

---

### Refund Errors

| Code | Name | Description |
|------|------|-------------|
| `30` | `RefundNotFound` | No refund record exists for the given refund ID. |
| `31` | `RefundAlreadyExists` | A refund with this ID has already been initiated. |
| `32` | `RefundWindowExpired` | The 30-day refund eligibility window has passed for this payment. |
| `33` | `RefundExceedsOriginal` | The requested refund amount would exceed the original payment amount. |
| `34` | `RefundNotApproved` | The refund must be in `Approved` status before it can be executed. |
| `35` | `RefundAlreadyCompleted` | The refund has already been executed or resolved. |
| `36` | `TooManyRefunds` | The number of pending refund requests for this order has reached the configured limit (`max_refunds_per_order`). |

---

### Refund Dispute Errors

| Code | Name | Description |
|------|------|-------------|
| `37` | `RefundNotRejected` | A dispute can only be raised on a `Rejected` refund. |
| `38` | `DisputeAlreadyExists` | A dispute for this refund ID has already been filed. |
| `39` | `DisputeNotFound` | No dispute record exists for the given refund ID. |

---

### Multisig Errors

| Code | Name | Description |
|------|------|-------------|
| `40` | `MultisigNotFound` | No multisig payment exists for the given payment ID. |
| `41` | `MultisigAlreadySigned` | This signer has already submitted a signature for this multisig payment. |
| `42` | `MultisigAlreadyExecuted` | The multisig payment has already been executed. |
| `43` | `InsufficientSignatures` | The number of collected signatures has not yet reached the required threshold. |

---

### Subscription Errors

| Code | Name | Description |
|------|------|-------------|
| `60` | `SubscriptionPlanAlreadyExists` | A subscription plan with this ID already exists. |
| `61` | `SubscriptionAlreadyExists` | A subscription with this ID already exists. |
| `62` | `SubscriptionNotFound` | No subscription exists for the given subscription ID. |
| `63` | `SubscriptionPlanNotFound` | No subscription plan exists for the given plan ID. |
| `64` | `SubscriptionNotActive` | The subscription is not in `Active` status. |
| `65` | `SubscriptionMaxCyclesReached` | The subscription has completed all configured billing cycles. |
| `66` | `SubscriptionIntervalNotElapsed` | The billing interval has not yet elapsed since the last charge. |

---

### General Errors

| Code | Name | Description |
|------|------|-------------|
| `50` | `InvalidInput` | One or more input fields are empty, too long, or otherwise invalid. |
| `51` | `PaginationLimitExceeded` | The requested page size is 0 or exceeds the maximum of 100. |
| `52` | `BatchSizeExceeded` | A `batch_payment` call contained more than 10 items. |
| `53` | `InvalidTags` | Tags array has more than 5 items, or an individual tag is empty or exceeds 32 characters. |
| `54` | `InvalidNonce` | The provided nonce does not match the stored per-payer nonce for replay protection. |

---

## SDK Usage

Errors are exposed in the SDK via `sdk/src/errors.ts`:

```typescript
import { LumenFlowError, PaymentErrorCode } from './errors';

try {
  await contract.createPaymentRequest({ ... });
} catch (e) {
  if (e instanceof LumenFlowError) {
    if (e.code === PaymentErrorCode.TokenNotAllowed) {
      console.error('Token not whitelisted. Ask admin to add it.');
    }
  }
}
```

---

*For event documentation, see [events-reference.md](events-reference.md).*  
*For auth model, see [auth-model.md](auth-model.md).*
