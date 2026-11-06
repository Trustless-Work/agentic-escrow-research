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

## Evidence updates since this RFC

Later repository evidence shows the baseline protected-payment lifecycle is no longer only hypothetical. Two different release-condition implementations now exist:

- [Evidence-gated milestone release testnet pilot](../research-notes/evidence-gated-release-testnet-pilot-2026-10-14.md): an automated approver checks anchored evidence before approve/release on a Multi-Release escrow, and refuses bad hashes, missing attestations, and wrong provider keys. Testnet only; decision-step code is private; evidence integrity is not proof of truth.
- [Contract addresses as TW escrow roles](../research-notes/contract-addresses-as-tw-escrow-roles-testnet-2026-11-02.md): receipt-gated release and deadline refund on unmodified single-release Trustless Work wasm, with contracts holding escrow roles. Testnet only; multi-release untested.

Related curated lesson: [settlement choice](../learnings/settlement-choice.md), [bounded authority](../learnings/bounded-authority.md).

The original Open Questions above remain preserved. They are partly informed, not closed.

## Remaining design decision

What lifecycle and binding-identity semantics should every protected-payment implementation share across release policies—order/escrow correlation, fulfillment-evidence binding, approval versus signing, expiration/refund paths, and final-state inspection—without treating any one pilot as production-ready?
