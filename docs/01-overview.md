# 01 – Project Overview

## Objective

Design a MiCA-compliant stablecoin-enabled EU neobank infrastructure.

The system must:

- Support SEPA Instant
- Maintain safeguarded fiat accounts
- Enable 1:1 EUR stablecoin minting and redemption
- Integrate compliance (MiCA + AMLD + DORA)
- Maintain operational resilience

---

## Core Design Question

Should the EMI:

A) Issue its own MiCA-compliant EUR stablecoin  
B) Integrate an existing issuer (e.g., EURC)

This decision affects licensing, reserve management, and risk exposure.
