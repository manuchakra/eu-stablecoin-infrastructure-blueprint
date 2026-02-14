# 02 – Scope & Assumptions

## Product Scope (V1)

We design an EU EMI that offers:

- EUR accounts with SEPA Instant transfers
- Fiat ledger (double-entry, idempotent posting)
- Custodial stablecoin wallet (EUR EMT under MiCA)
- Fiat ↔ Stablecoin 1:1 conversion (mint/redeem)
- Compliance layers (AMLD/Travel Rule concepts)
- Operational resilience considerations (DORA)

## What This Is NOT (V1)

- Lending / credit underwriting
- High-frequency trading / market making
- Full bank license operations
- Consumer app UX deep dive

## Key Assumptions

- EMI holds client funds in safeguarded accounts at partner bank(s)
- Stablecoin side is MiCA compliant (EMT model)
- Custody is either in-house with strong controls or via regulated custodian
- System is designed for auditability and failure recovery
