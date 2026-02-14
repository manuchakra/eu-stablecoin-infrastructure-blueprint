# 06 – Fiat ↔ Stablecoin Conversion Flows (Integration Model)

## Objective

Design 1:1 EUR fiat ↔ EUR stablecoin conversions for an EU EMI that:
- holds client funds in safeguarded bank accounts
- integrates an external MiCA-compliant stablecoin issuer (Option A)
- provides custodial wallets for users

This document focuses on:
- conversion states
- ledger postings
- issuer interactions
- failure handling
- reconciliation requirements

---

## Reserve Model (V1)

### Approach
Fiat stays in the safeguarded pool account(s) at partner bank(s), but is **earmarked** in the ledger:

- **Customer Liability Accounts**: represent user fiat balances
- **Conversion Hold Account**: temporary state during processing
- **Stablecoin Backing Earmark Account**: tracks fiat reserved to back stablecoin supply

This creates auditable separation without requiring separate bank accounts in V1.

---

## Fiat → Stablecoin (Mint) Flow

### Goal
User converts €X fiat into €X stablecoin in their custodial wallet.

### Steps (Happy Path)
1. User requests conversion: fiat → stablecoin (amount X).
2. API validates request + authenticates user.
3. AML checks run (real-time):
   - user risk
   - transaction limits
   - behavioral signals
4. Ledger checks available fiat balance.

#### Ledger: place funds on hold
5. **Debit** Customer Liability (User Fiat) X
6. **Credit** Conversion Hold X
7. Transaction state = Pending

#### Call issuer to mint
8. Stablecoin Integration Service sends mint request to Issuer:
   - amount
   - destination wallet address (custodial)
   - reference id (transaction_id / idempotency key)
9. Issuer returns acceptance (async) or immediate confirmation (depends on issuer API).

#### On-chain mint confirmed
10. On-chain mint confirmation detected (webhook / polling / event listener).
11. Custodial wallet balance updated (stablecoin credited).

#### Ledger: move hold to earmarked backing
12. **Debit** Conversion Hold X
13. **Credit** Stablecoin Backing Earmark X
14. Transaction state = Settled

---

## Mint Failure Handling

### If issuer rejects mint (compliance / technical)
1. Record rejection reason.
2. Reverse hold:
   - **Debit** Conversion Hold X
   - **Credit** Customer Liability X
3. Transaction state = Failed

### If mint is "unknown" (timeouts)
1. Transaction state = Unknown
2. Do NOT release hold immediately (prevents double spend)
3. Reconcile with issuer:
   - status inquiry using transaction_id
4. If later confirmed minted:
   - settle (steps 12–14)
5. If later confirmed not minted:
   - reverse hold

---

## Stablecoin → Fiat (Redeem) Flow

### Goal
User converts €X stablecoin into €X fiat in EMI ledger.

### Steps (Happy Path)
1. User requests redemption: stablecoin → fiat (amount X).
2. API validates request + authenticates user.
3. AML checks run (wallet origin, risk).
4. Custodial wallet balance checked.

#### Freeze stablecoin amount internally
5. Mark stablecoin amount as locked for redemption (prevents double send).

#### Burn via issuer
6. Send burn request to issuer:
   - amount X
   - source wallet address
   - idempotency key = transaction_id
7. Issuer processes burn on-chain.

#### On-chain burn confirmed
8. On-chain burn confirmation received.
9. Update custodial wallet balance.

#### Ledger: release earmarked fiat back to user
10. **Debit** Stablecoin Backing Earmark X
11. **Credit** Customer Liability (User Fiat) X
12. Transaction state = Settled

---

## Redemption Failure Handling

### If burn fails / rejected
- Unlock stablecoin
- Mark transaction failed
- No fiat credited
- Full audit log retained

### If burn is unknown (timeout)
- Keep stablecoin locked
- Reconcile with issuer status inquiry
- Resolve based on confirmed on-chain state

---

## Idempotency Rules (Critical)

### Idempotency Key = transaction_id
All mint/burn requests must be idempotent across:
- EMI internal services
- issuer API calls
- wallet updates

If a duplicate request arrives:
- Return the previously stored outcome
- Do not re-post ledger entries
- Do not re-trigger mint/burn

---

## Reconciliation & Auditability

Daily reconciliation must cover:

1. **Stablecoin backing**
   - Sum(Stablecoin Backing Earmark)
   must equal
   - Total stablecoin in custody (for users) tracked by wallet engine

2. **Safeguarded pool**
   - Bank statement balance
   must align with
   - Total customer liabilities + earmarked backing (per design)

3. **Issuer confirmations**
   - All minted/burned events must map to internal transaction_ids

Any mismatch triggers:
- incident workflow
- temporary conversion freeze
- investigation + corrective postings

---

## Key Risks

- Issuer dependency risk
- Custody/key management failure
- Depeg / issuer reserve problems (monitoring needed)
- Race conditions (hold vs mint confirmations)
- Ledger/issuer mismatch

---

## Future Enhancements (v2)

- Separate bank reserve accounts for stablecoin backing
- Multi-issuer routing
- Real-time proof-of-reserves ingestion
- Automated depeg mitigation policy
