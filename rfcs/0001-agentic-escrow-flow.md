# RFC 0001: Baseline Agentic Escrow Flow

Status: Draft

## Summary

This RFC proposes a baseline flow for an agent-mediated transaction protected by escrow.

The goal is to define the smallest useful pattern that demonstrates the difference between direct agent payment and conditional settlement.

## Flow

```text
Human defines intent
Buyer Agent searches or negotiates
Seller Agent or merchant confirms terms
Escrow is created
Human or policy wallet funds escrow
Merchant fulfills
Evidence is submitted
Approver approves or disputes
Release signer releases funds
Final state is inspectable
```

## Actors

| Actor | Role |
| --- | --- |
| Human principal | Defines intent and may fund or approve |
| Buyer Agent | Orchestrates discovery, terms, and state monitoring |
| Seller Agent | Presents offer, confirms order, submits status |
| Merchant / provider | Receives payment after fulfillment |
| Approver | Verifies completion or rejects evidence |
| Release signer | Performs release when conditions are met |
| Dispute resolver | Handles contested outcomes |

## Why Escrow

Direct payment makes the economic action final immediately. Escrow allows the buyer side to commit funds while preserving conditional release.

This supports:

- payment commitment before fulfillment;
- explicit fulfillment criteria;
- approval or dispute before finality;
- auditability of intent, evidence, and state;
- bounded authority for agents.

## Minimal Implementation Target

An initial implementation should support:

- one order;
- one escrow;
- one protected payment;
- one fulfillment evidence event;
- one approval or dispute path;
- one release or refund outcome.

## Non-Goals

- general-purpose agent marketplace;
- fully autonomous settlement;
- new escrow contract behavior;
- production-ready merchant network;
- removing the human authorization boundary too early.

## Open Questions

- Should the agent create the escrow directly or prepare it for human confirmation?
- Should funding always remain human-signed in V1?
- Should approval and release be separate roles?
- What minimum evidence should be required before release?
- How should the order identifier bind to the escrow?
