# 07 – MiCA Compliance Mapping (EMT Integration Model)

## Objective

Map MiCA requirements for E-Money Tokens (EMTs) 
to our EMI + Stablecoin Integration architecture.

This document assumes:

- The stablecoin is an EMT denominated in EUR.
- The issuer is an external, MiCA-authorised entity.
- Our EMI integrates the EMT via API.
- We provide custodial wallet infrastructure to users.

---

# 1. EMT Issuer Requirements (External Entity)

Under MiCA, EMT issuers must:

- Be authorised as a credit institution or EMI.
- Maintain 1:1 backing with reserves.
- Provide clear redemption rights.
- Publish whitepaper disclosures.
- Implement governance and risk controls.

### Our Exposure

We must assess:

- Issuer authorisation status.
- Reserve transparency.
- Redemption guarantees.
- Operational reliability.
- Regulatory standing.

This is a **third-party risk dependency**.

---

# 2. Redemption Rights (Critical for EMTs)

MiCA requires:

- Token holders must have the right to redeem at par value.
- Redemption must occur at any time.
- No unfair fees.

### Architectural Mapping

Our system supports:

- Stablecoin → Fiat redemption flow.
- Immediate ledger credit after burn confirmation.
- Clear transaction states.
- Audit logs for redemption history.

---

# 3. Safeguarding & Reserve Transparency

Although the issuer manages reserves, our EMI must:

- Track earmarked fiat backing.
- Maintain daily reconciliation.
- Monitor issuer reserve attestations.

### Controls Implemented

- Stablecoin Backing Earmark Account in ledger.
- Daily reconciliation against:
  - Custodial wallet balances
  - Issuer supply confirmations
- Incident trigger on mismatch.

---

# 4. Custody & Key Management

MiCA requires strong operational controls.

If we provide custodial wallets:

We must ensure:

- Secure key storage (MPC / HSM).
- Access control segregation.
- Incident response playbooks.
- Disaster recovery capability.

This intersects with DORA obligations.

---

# 5. Governance & Risk Management

Although issuer-level governance is external,
our EMI must implement:

- Third-party risk assessment framework.
- Counterparty exposure monitoring.
- Depeg monitoring policy.
- Conversion suspension policy (if systemic risk detected).

---

# 6. Whitepaper & Disclosure Transparency

MiCA requires transparency around:

- Token risks
- Reserve structure
- Redemption mechanisms

As an integrating EMI, we must:

- Clearly disclose that we do not issue the token.
- Identify the authorised issuer.
- Explain conversion mechanics.
- Explain custody model.

---

# 7. AML & Travel Rule Alignment

Although EMT is regulated, AML obligations remain.

Our system must:

- Perform KYC on all users.
- Monitor on-chain activity (blockchain analytics integration).
- Apply Travel Rule compliance (where applicable).
- Block sanctioned addresses.

---

# 8. Operational Resilience (MiCA + DORA Intersection)

We must manage:

- Issuer API outage risk.
- Blockchain congestion.
- Custody system failure.
- Depeg scenarios.

Resilience strategy includes:

- Conversion hold states.
- Unknown transaction handling.
- Incident freeze procedures.
- Manual override governance.

---

# 9. Key Risk Summary

| Risk | Mitigation |
|------|------------|
| Issuer insolvency | Due diligence + diversification (future) |
| Reserve opacity | Monitor attestations |
| Depeg event | Automated monitoring + conversion suspension |
| Custody breach | MPC/HSM + strict access control |
| Regulatory change | Continuous compliance review |

---

# Conclusion

By integrating a MiCA-compliant EMT issuer and 
implementing strong ledger segregation, reconciliation, 
custody controls, and AML integration, 

the EMI can provide stablecoin services while maintaining 
regulatory alignment and operational resilience.
