# 05 – SEPA Instant Processing Flow

## Objective

Design a robust SEPA Instant payment processing lifecycle
with idempotency, failure handling, and reconciliation.

---

## High-Level Requirements

- 24/7 availability
- ≤10 second end-to-end response
- Idempotent transaction handling
- Real-time ledger updates
- Clear failure rollback logic

---

## Outbound SEPA Instant Flow (User Sends Money)

### Step 1 – User Initiation
1. User initiates transfer.
2. API Gateway authenticates request.
3. Request validated (IBAN format, amount limits).

### Step 2 – Pre-Checks
4. Ledger checks available balance.
5. AML engine runs real-time risk scoring.
6. Sanctions screening executed.
7. Rate limits checked.
8. Transaction ID generated (idempotency key).

### Step 3 – Ledger Hold
9. Debit Customer Liability account (Pending state).
10. Credit "Outgoing SEPA Pending" account.
11. Transaction state = Pending.

### Step 4 – Rail Submission
12. SEPA message (pacs.008) generated.
13. Sent to clearing mechanism (RT1 / TIPS).
14. Await acknowledgment.

### Step 5 – Response Handling

**If Accepted (≤10s):**

15. Confirmation received.
16. Pending entry marked Settled.
17. Funds removed from Safeguarded Pool.
18. Outgoing SEPA Pending account cleared.
19. Transaction state = Settled.

**If Rejected:**

20. Rejection reason logged.
21. Reverse ledger hold:
    - Debit Outgoing Pending
    - Credit Customer Liability
22. Transaction state = Failed.

**If Timeout (No response in SLA):**

23. Mark transaction as "Unknown".
24. Trigger reconciliation queue.
25. Retry status inquiry (pacs.002).
26. If later confirmed:
    - Settle normally.
27. If confirmed failed:
    - Reverse hold.

---

## Inbound SEPA Instant Flow (User Receives Money)

1. Incoming message received.
2. Validate message integrity.
3. AML screening (beneficiary).
4. Credit Safeguarded Funds account.
5. Credit Customer Liability account.
6. Mark transaction Settled.
7. Notify user in real-time.

---

## Idempotency Design

Every request includes:

- transaction_id
- client_id
- timestamp
- hash signature

Duplicate requests:
→ Return original transaction result
→ Do not double post ledger entries

---

## Key Risks

- Duplicate posting
- Race conditions during balance checks
- Timeout ambiguity
- Clearing network outage
- Ledger/rail mismatch

---

## Reconciliation Strategy

Daily:

1. Compare outgoing transactions vs clearing confirmations.
2. Compare safeguarded bank balance vs ledger sum.
3. Auto-flag discrepancies.

---

## Design Tradeoffs

- Strong consistency vs performance
- Synchronous confirmation vs async queue model
- Rail dependency risk
