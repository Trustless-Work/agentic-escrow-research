# Enterprise Procurement

## Summary

An enterprise agent helps source, order, and pay vendors under internal policy constraints. Escrow protects the payment while preserving approvals, auditability, and recovery.

## Actors

- Company principal
- Procurement agent
- Vendor or vendor agent
- Finance approver
- Compliance reviewer
- Release signer
- Dispute resolver

## Flow

```text
Company defines procurement policy
Agent discovers vendor or quote
Vendor confirms terms and invoice
Escrow is prepared or created
Finance funds or approves funding
Vendor performs
Evidence is reviewed
Funds release, dispute, or refund
```

## Why Direct Payment May Be Insufficient

Enterprises need controls around budget, vendor approval, compliance, invoice matching, and payment release. Giving an agent a funded wallet does not satisfy those controls.

## Escrow Value

- policy-constrained procurement;
- invoice-to-escrow binding;
- auditable approvals;
- vendor confidence that funds are committed;
- buyer confidence that release is conditional.

## Authority Model

The agent may handle sourcing, comparison, and documentation. Finance or policy-controlled wallets should approve funding and release until stronger safeguards exist.

## Failure Modes

- unauthorized vendor;
- invoice mismatch;
- duplicate payment;
- vendor nonperformance;
- compliance issue after funding;
- approver unavailable;
- agent exceeds mandate.

## Open Questions

- How should enterprise policy map to escrow roles?
- Can ERP or procurement systems provide authoritative invoice evidence?
- What approval thresholds should apply by amount and vendor type?
