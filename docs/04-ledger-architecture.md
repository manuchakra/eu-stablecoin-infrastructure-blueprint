# 04 – Fiat Ledger Architecture

## Objective

Design a double-entry ledger system that:

- Maintains accurate client balances
- Supports pending and settled states
- Is idempotent (safe against duplicate requests)
- Is auditable
- Reconciles against safeguarded accounts

The ledger is the source of truth for fiat balances.

---

## Core Design Principle: Double-Entry Accounting

Every transaction must:

- Debit one account
- Credit another account

Total debits must always equal total credits.

The ledger must never be allowed to go out of balance.

---

## Account Types

### 1. Customer Liability Accounts
- Represent client balances
- From accounting perspective: liabilities of the EMI

### 2. Safeguarded Funds Account
- Represents total client funds held at partner bank
- Must reconcile daily with external bank statement

### 3. Conversion Hold Account
- Temporary account when converting fiat → stablecoin
- Prevents double spending during mint process

### 4. Fee Revenue Account (future use)
- For transaction fees

---

## Transaction States

Each transaction must have a lifecycle:

1. Created
2. Pending
3. Settled
4. Reversed (if needed)
5. Failed

No direct balance mutation without state tracking.

---

## Example: SEPA Incoming Transfer

1. Bank confirms incoming €1,000.
2. Debit: Safeguarded Funds Account €1,000
3. Credit: Customer Liability Account €1,000
4. Transaction marked as Settled.

Ledger remains balanced.

---

## Example: Fiat → Stablecoin Conversion

User converts €500.

Step 1:
- Debit: Customer Liability €500
- Credit: Conversion Hold €500

Step 2:
- Mint request sent to issuer.

If mint succeeds:
- Debit: Conversion Hold €500
- Credit: Safeguarded Funds Reserved €500

Stablecoin minted.

If mint fails:
- Reverse Step 1.

---

## Idempotency Requirement

Every transaction must include:

- Unique transaction ID
- Duplicate detection logic

If the same request is retried:
→ System must not double-post.

---

## Reconciliation Requirements

Daily checks:

1. Sum of all Customer Liability Accounts
   must equal
2. Safeguarded Funds Balance

If mismatch:
- Trigger incident
- Freeze outbound transactions
- Investigate discrepancy

---

## Key Risks

- Double posting
- Ledger drift
- Race conditions
- Incorrect state transitions
- Reconciliation mismatch

---

## Design Tradeoffs

- Event-sourced ledger vs balance-updating ledger
- Real-time settlement vs batch posting
- Strong consistency vs eventual consistency

---

## Event-Sourced Ledger Model (Recommended)

### Core Tables / Objects

1. **Journal Entries**
   - Immutable rows (append-only)
   - Each row has:
     - entry_id
     - transaction_id (idempotency key)
     - timestamp
     - debit_account
     - credit_account
     - amount
     - currency
     - state (pending/settled/reversed)
     - metadata (rail, counterparty, reference)

2. **Transaction Record**
   - Groups multiple journal entries into a single logical transaction
   - Tracks lifecycle state transitions

3. **Balance Views (Derived)**
   - Computed by summing journal entries per account
   - Cached in a materialized view for performance

### Why This Helps Regulated Fintech

- Full audit trail (no hidden mutations)
- Easy reconciliation and dispute investigation
- Can rebuild balances from scratch if needed
- Strong safety against silent corruption

### Performance Strategy

- Write path: append journal entry + update cached balance (optional)
- Read path: read cached balance + verify against journal periodically
- Nightly batch: recompute balances from journal as integrity check
