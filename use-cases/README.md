# Use Cases

This folder collects concrete agentic escrow use cases.

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
