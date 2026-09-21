# Outcome Agreement Protocols - 2026-09-21

Status: Research reference  
Scope: job escrow, agreement escrow, verification, and recourse protocols  
Normative status: non-normative; does not rewrite or approve any RFC

## Purpose

This note collects references for systems that are closer to outcome-conditioned settlement than to immediate payment or simple delayed capture.

The key research question is whether Trustless Work should be interpreted as a minimal payment hold, a job escrow, an agreement-settlement primitive, or a milestone graph for programmable agreements.

## Reference Map

| Source | What it contributes | Relationship to Trustless Work |
| --- | --- | --- |
| ERC-8183 | Job escrow with Client, Provider, Evaluator, funding, submission, completion, rejection, and expiry | Closest minimal protocol analogue |
| Arc ERC-8183 tutorial | Demonstrates an agentic economic flow around ERC-8183 concepts | Ecosystem validation; deployment details require separate verification |
| VCAP draft | Negotiation, escrow hold, delivery, independent verification, release/refund, escalation | Potential adapter target; expects an external escrow/payment rail |
| VCAP-related drafts | AIVS, ATEP, ATXN, and AP2 binding material around verification, exchange, and signed transaction elements | Useful design corpus; individual drafts, not consensus standards |
| ASP paper | Stablecoin authorize/capture escrow with deadlines, fulfillment verification, laddering, and partial refunds | Adjacent delayed-commerce architecture |
| Sinetti / FNA trust-layer framing | Frames identity, escrow, recourse, and reputation as a trust layer for agentic payments | Market/protocol direction; implementation maturity requires validation |
| ENACT | Agent job escrow and tooling signal | Direct product signal; implementation depth requires validation |

## ERC-8183 Comparison Handle

Approximate role mapping:

| ERC-8183 | Trustless Work |
| --- | --- |
| Client | Funder |
| Provider | Receiver |
| Evaluator | Approver |

Trustless Work additionally separates:

- Milestone Marker;
- Release Signer;
- Dispute Resolver;
- Platform;
- multiple milestones and progressive settlement in Multi Release.

Working distinction to test:

> ERC-8183 is a minimal job primitive. Trustless Work may be an agreement-settlement primitive.

## VCAP Adapter Handle

Possible conceptual mapping:

| VCAP concept | Trustless Work concept to test |
| --- | --- |
| `escrow_hold` | deploy + fund escrow |
| `service_delivery` | milestone status / fulfillment evidence |
| `verification_callback` | approve or dispute |
| settlement / escalation | release, refund, or resolve dispute |

Current gaps to analyze:

- external verifier semantics;
- evidence/proof provenance;
- proof signatures;
- order-to-escrow binding;
- safe signing for automated release;
- whether VCAP requires escrow behavior that TW does not expose today.

## References

- <https://eips.ethereum.org/EIPS/eip-8183>
- <https://www.arc.io/blog/running-an-agentic-economic-flow-on-arc-with-erc-8183>
- <https://datatracker.ietf.org/doc/html/draft-stone-vcap-02>
- <https://arxiv.org/abs/2609.02208>
- <https://fna.fi/archive/insights/agentic-payments-trust-layer>
- <https://www.enact.info/>

## Analysis To Do

- Compare TW, ERC-8183, and CPP/x402 `auth-capture` state-by-state and function-by-function.
- Decide whether a VCAP-to-TW adapter can be prototyped above existing contracts.
- Separate protocol design material from deployed production proof.
