# Settlement choice: direct payment versus escrow

**ID:** `settlement/direct-vs-escrow`  
**Status:** research-supported  
**Confidence:** limited  
**Related RFCs:** [0003](../rfcs/0003-x402-escrow-composition.md), also informs [0001](../rfcs/0001-agentic-escrow-flow.md) and [0004](../rfcs/0004-agent-facing-tool-schema.md)  
**Example:** [../examples/direct-vs-escrow.json](../examples/direct-vs-escrow.json)

## Question

When should an agent use a direct machine payment, and when should it protect a commitment with escrow?

## Lesson

Direct payment and escrow are complementary.

- Use **direct payment** for small, immediate, digital access where fulfillment is granted at request time and there is no meaningful condition left to protect after settlement.
- Use **escrow** when fulfillment is delayed, quality is uncertain, the amount is meaningful, evidence must be reviewed, or dispute, timeout, or refund paths matter.
- Treat **payment-infrastructure risk** as a second axis. Escrow addresses counterparty and fulfillment risk. It does not automatically solve facilitator outages, ambiguous settlement, transport failures, or dependency misconfiguration.

Principle supported by the use-case evidence:

> Do not introduce conditional settlement when there is no meaningful condition to protect.

A complete end-to-end **x402 access payment → Trustless Work escrow for protected principal** composition has been discussed and partially evidenced on each side, but **not demonstrated as one integrated flow**.

## Evidence

| Claim | Class | Source |
| --- | --- | --- |
| Small immediate digital purchase can settle directly without escrow value | mainnet | [Nirium direct x402 payment](../use-cases/nirium-direct-x402-payment.md) |
| Direct x402 and TW multi-release milestone payouts both work, but are not yet composed | mainnet / production-tested use case | [Nirium bounded milestone payouts](../use-cases/nirium-bounded-milestone-payouts.md) |
| Direct vs escrow decision rule and composition pattern | RFC draft + evidence sections | [RFC 0003](../rfcs/0003-x402-escrow-composition.md) |
| Who submits the transaction changes the failure mode; settled ≠ delivered | testnet; one historical push failure is unverifiable | [x402 vs MPP Charge](../research-notes/x402-vs-mpp-charge-testnet-2026-10-21.md) |
| Access / usage / immediate response vs outcome needing evidence or dispute | adjacent Stellar evidence | [Open Stellar skills marketplace](../use-cases/open-stellar-skills-marketplace.md), [Open Stellar payment evidence](../research-notes/open-stellar-agentic-payment-evidence-2026-09-21.md) |

## Limitations

- No demonstrated combined x402 → escrow composition with shared identifiers and receipts.
- MPP Charge settlements in the comparison note are testnet only; the note states MPP Charge has never settled on mainnet.
- The September 2026 push-mode “paid and not delivered” claim is labelled **unverifiable** in its source note: only a hash prefix survives.
- Decision tables here are research guidance, not product policy.

## Remaining design decision

When a flow contains both rails, what shared identifiers and receipts connect the access payment, the escrow agreement, fulfillment evidence, and recovery?
