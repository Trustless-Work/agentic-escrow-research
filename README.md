# Agentic Escrow Research

Open-source research on conditional settlement, bounded authority, and dispute-aware payments for the agentic economy.

## Thesis

AI agents will increasingly discover, negotiate, purchase, hire, and coordinate economic activity. Stablecoins are a strong settlement medium for that world, but direct payments and agent wallets alone do not answer the core trust question:

> Under what conditions should a payment become final?

A wallet spending limit constrains how much an agent can spend. Agentic escrow constrains when, why, and under what verified conditions committed funds can be released.

Trustless Work is exploring escrow as a machine-readable trust layer for humans, agents, merchants, autonomous services, and platforms.

## What This Repository Is

This repository is a public research initiative. It is meant to host:

- a living whitepaper;
- open RFCs;
- use-case analysis;
- diagrams and trust models;
- reference experiments;
- implementation notes;
- contributor research questions.

It is not a production protocol commitment. Research conclusions should be validated before they are promoted into Trustless Work's core product, API, SDK, MCP, or smart-contract roadmap.

## Starting Points

| File | Purpose |
| --- | --- |
| [CHARTER.md](CHARTER.md) | Initiative scope, principles, and governance |
| [WHITEPAPER.md](WHITEPAPER.md) | Living whitepaper v0 |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How contributors can participate |
| [rfcs/0001-agentic-escrow-flow.md](rfcs/0001-agentic-escrow-flow.md) | Baseline agentic escrow flow |
| [rfcs/0002-agent-authority-levels.md](rfcs/0002-agent-authority-levels.md) | Human/agent autonomy model |
| [rfcs/0003-x402-escrow-composition.md](rfcs/0003-x402-escrow-composition.md) | x402 and escrow composition |
| [rfcs/0004-agent-facing-tool-schema.md](rfcs/0004-agent-facing-tool-schema.md) | Candidate tool/API schema for agents |
| [use-cases/](use-cases/) | Use-case library and implementation evidence |
| [use-cases/nirium-bounded-milestone-payouts.md](use-cases/nirium-bounded-milestone-payouts.md) | Real-world bounded-authority milestone payout example |
| [use-cases/nirium-treasury-missing-parameter.md](use-cases/nirium-treasury-missing-parameter.md) | Authority-by-missing-parameter treasury example |

## Research Tracks

1. Agentic commerce: agents buying goods, services, software, data, or compute.
2. Agent-to-agent work: agents hiring other agents or autonomous service providers.
3. Enterprise procurement: bounded agents coordinating vendor payments, invoices, and approvals.
4. Bounties and grants: agents matching work, evidence, approvals, and milestone payments.
5. Machine-native APIs: x402-style direct payments combined with conditional escrow for larger or higher-risk commitments.
6. Safety and authorization: signing policy, key custody, approval separation, disputes, recovery, and recourse.
7. Bounded control of owned capital: recurring automated actions where capability design constrains what economic outcomes an agent can express.

## Core Questions

- What should an agent be allowed to prepare, create, fund, approve, dispute, and release?
- Which economic actions require human approval in early implementations?
- How should order, invoice, evidence, and escrow state bind together?
- When is a direct machine payment enough?
- When does conditional settlement provide materially better safety?
- What tool schema should make invalid authority obvious before transaction submission?
- What failure paths are required if an agent, merchant, oracle, verifier, or user disappears?
- Which dangerous economic actions can be made structurally inexpressible to an automated actor?
- How should we distinguish exfiltration authority from economic-decision authority?

## Current Working Model

The recommended starting architecture keeps the escrow primitive narrow and composes agentic behavior through APIs, SDKs, MCP-style tools, policies, and orchestration.

```text
Human intent
  -> buyer agent negotiation
  -> merchant/agent confirmation
  -> protected escrow commitment
  -> human or policy-controlled funding
  -> fulfillment evidence
  -> approval / dispute / fallback
  -> release or refund
```

Implementation evidence is beginning to refine this model. The [Nirium bounded-authority milestone payout use case](use-cases/nirium-bounded-milestone-payouts.md) demonstrates a concrete pattern in which a human fixes the economic commitment up front while automation is limited to state transitions inside already-created, allowlisted escrows.

> **Human defines the economic boundary; automation operates inside that boundary.**

A second Nirium implementation adds another form of bounded authority: [authority by missing parameter](use-cases/nirium-treasury-missing-parameter.md). In this pattern, the automated actor's callable operation does not expose an arbitrary beneficiary parameter.

> **Make unauthorized economic actions inexpressible, not merely disallowed.**

Together, these examples suggest that safe agentic finance depends both on **pre-authorized objects an agent may act upon** and on **capabilities the agent is never given in the first place**.

## Relationship to Trustless Work

Trustless Work provides stablecoin escrow infrastructure. This research explores how that infrastructure could support agentic commerce without giving agents unchecked financial authority.

The research should prefer:

- non-custodial user authority;
- explicit role separation;
- deterministic escrow state;
- narrow smart-contract primitives;
- machine-readable workflows;
- bounded agent authorization;
- recoverable human control;
- clear dispute and fallback paths.

## Status

Early public research. Contributions, critiques, diagrams, use cases, and experiments are welcome.
