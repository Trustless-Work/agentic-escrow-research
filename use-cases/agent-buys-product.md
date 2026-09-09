# Agent Buys a Product

## Summary

A human asks an agent to buy a product within a budget. The agent finds a merchant, negotiates or selects terms, and creates a protected payment. Funds release only after fulfillment is verified.

## Actors

- Human buyer
- Buyer agent
- Merchant or seller agent
- Merchant receiver
- Approver or verification policy
- Release signer

## Flow

```text
Human defines intent and budget
Buyer agent selects product and merchant
Seller confirms order terms
Escrow is created
Human funds escrow
Merchant ships or delivers
Evidence is posted
Buyer side approves or disputes
Funds release or refund path starts
```

## Why Direct Payment May Be Insufficient

The agent may select the wrong product, misunderstand delivery terms, or pay before the merchant fulfills. A spending limit only caps loss size; it does not create recourse.

## Escrow Value

- payment commitment without immediate finality;
- fulfillment-linked release;
- inspectable order and escrow state;
- dispute path if the merchant fails;
- human-controlled funding in early versions.

## Authority Model

Initial version should let the agent prepare or create escrow terms while the human signs funding. Approval may remain human-controlled or policy-assisted.

## Failure Modes

- merchant never fulfills;
- product differs from intent;
- delivery evidence is ambiguous;
- buyer agent is manipulated;
- seller agent submits false status;
- human is unavailable after funding.

## Open Questions

- What evidence is enough for product delivery?
- Should shipping carrier data be required?
- Can a merchant agent be trusted to submit fulfillment evidence?
- When should the agent raise a dispute automatically?
