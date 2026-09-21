# Use Cases

This folder collects concrete use cases and implementation evidence relevant to agentic escrow and bounded economic authority.

Each use case should explain why direct payment may be insufficient and what conditional settlement adds.

The library includes both **conceptual use cases** and **implementation evidence** contributed by builders. Implementation evidence can inform the RFCs without becoming a production commitment.

## Current Use Cases

| Use case | Type | Core question |
| --- | --- | --- |
| [Agent buys a product](agent-buys-product.md) | Conceptual | How does an agent purchase without releasing funds before delivery? |
| [Agent purchases compute](agent-purchases-compute.md) | Conceptual | When is direct machine payment enough, and when does escrow help? |
| [Agent hires another agent](agent-hires-agent.md) | Conceptual | How can autonomous services pay each other for outcomes? |
| [Enterprise procurement](enterprise-procurement.md) | Conceptual | How can company agents coordinate controlled vendor payments? |
| [Grants and bounties](grants-and-bounties.md) | Conceptual | How can agents help route milestone-based capital? |
| [Nirium: bounded-authority milestone payouts + x402](nirium-bounded-milestone-payouts.md) | Implementation evidence | How can a human define the economic commitment while automation is limited to approving and releasing already-authorized milestone payouts? |
| [Nirium: authority by missing parameter](nirium-treasury-missing-parameter.md) | Implementation evidence | Can automated capital management be made safer by designing the callable operation so an arbitrary beneficiary cannot be expressed? |
| [Nirium: direct x402 payment without escrow](nirium-direct-x402-payment.md) | Implementation evidence | When is direct payment sufficient, and what failure modes remain even when escrow would add no value? |
| [Open Stellar skills marketplace](open-stellar-skills-marketplace.md) | Adjacent implementation evidence | When should an agent-paid skill use direct Stellar x402 payment, and when should it become escrow-backed outcome settlement? |

## Implementation Evidence

### Nirium: bounded-authority milestone payouts

[Nirium's contribution](nirium-bounded-milestone-payouts.md) documents a real Trustless Work Multi-Release integration in which:

- a human creates the escrow and fixes milestones, amounts, receivers, and roles;
- an automated signer can approve and release milestones only on allowlisted, already-created escrows;
- the automated signer cannot choose a new receiver or create a new economic commitment;
- dispute resolution remains outside the automated path;
- x402 is used separately for direct, immediate payments and is explored as a complementary primitive to escrow.

The main research pattern is:

> **Human defines the economic boundary; automation operates inside that boundary.**

This is evidence for bounded agent authority, not a commitment to a specific future Trustless Work API, signing architecture, or x402 composition.

### Nirium: authority by missing parameter

[Nirium's treasury contribution](nirium-treasury-missing-parameter.md) documents a different bounded-authority pattern. This flow does not use escrow: a policy-constrained RebalanceManager operates on a client-owned DeFindex vault, while the callable `rebalance()` operation does not expose an arbitrary external recipient parameter.

The reusable research principle is:

> **Make unauthorized economic actions inexpressible, not merely disallowed.**

This introduces a useful distinction between two kinds of authority:

- **exfiltration authority** — can the automated actor direct capital to an arbitrary beneficiary?
- **economic decision authority** — can the automated actor make permitted decisions that still lose value through allocation, timing, slippage, or churn?

Restricting arbitrary beneficiaries can strongly reduce exfiltration risk without eliminating economic-decision risk.

This example is adjacent to escrow rather than an escrow flow itself. It broadens the research question from only *when should a payment become final?* to also include *which economic actions should be impossible for an automated actor to express?*

### Nirium: direct x402 payment without escrow

[Nirium's direct-payment contribution](nirium-direct-x402-payment.md) documents a real x402 settlement for a small, immediate digital purchase where escrow would add unnecessary coordination overhead.

The reusable research principle is:

> **Do not introduce conditional settlement when there is no meaningful condition to protect.**

The contribution also separates two different questions:

- **settlement-model risk** — does fulfillment require conditional release, approval, dispute, or recourse?;
- **payment-infrastructure risk** — can the facilitator or another dependency be unavailable, ambiguous, or require reconciliation?

A flow can correctly use direct payment while still needing health checks, retries, receipts, reconciliation, and machine-readable failure semantics.

Evidence note: `x402-foundation/x402#3148` should not be treated as corroboration of the specific rejected-then-settled incident described in the contribution. The issue discussion corrected the original framing. Its durable value for this research is the difficulty of exposing stable machine-readable rejection/recovery reasons to unattended clients.

### Open Stellar: skills marketplace direct payment vs escrow boundary

[Open Stellar's contribution](open-stellar-skills-marketplace.md) documents a Stellar-oriented agent skills marketplace where a direct x402-style payment can trigger an immediate skill invocation.

Its main research value is the settlement boundary:

> **Use direct payment for access, usage, attempts, and immediate responses; use escrow when the agent is buying an outcome that needs evidence, review, dispute, or recovery.**

This complements the Nirium direct-payment evidence by adding a marketplace and agent-to-agent service-discovery shape.