# Agentic Escrow Research Charter

## Purpose

This initiative exists to research how escrow can become a conditional settlement and authorization layer for the agentic economy.

The goal is to lead an open-source conversation around financial infrastructure for AI agents that can act economically without receiving unchecked payment authority.

## Background

Agentic commerce introduces new transaction patterns:

- humans delegating purchases to agents;
- agents negotiating with merchant agents;
- autonomous services charging for outcomes;
- agents hiring other agents or contractors;
- enterprise agents coordinating procurement;
- agents consuming paid APIs, compute, data, or software.

Stablecoins make settlement programmable and internet-native. However, direct stablecoin transfers are final by design. That is powerful for low-risk payments and dangerous when intent, fulfillment, identity, evidence, or authorization is uncertain.

## Core Hypothesis

Stablecoins provide machine-native settlement. Escrow provides conditional trust around that settlement.

Agentic escrow can help answer:

- who authorized the transaction;
- what was the agent allowed to do;
- what condition must be satisfied before funds release;
- who can approve, dispute, or recover funds;
- what evidence binds the payment to the intended outcome.

## Scope

In scope:

- agentic commerce flows;
- authority and signing models;
- human-in-the-loop authorization;
- agent-to-agent transactions;
- escrow state machines;
- evidence and fulfillment models;
- dispute, refund, cancellation, and recovery paths;
- x402 and escrow composition;
- agent-facing API, SDK, MCP, and tool-schema design;
- reference experiments and implementation sketches.

Out of scope for the initial research:

- modifying audit-bound Trustless Work contracts;
- promising production autonomous commerce;
- adding contract complexity before validated need;
- treating any one demo as the final product direction;
- custody models that hide material economic authority from users.

## Principles

1. Keep the escrow primitive narrow.
2. Separate agent orchestration from settlement authority.
3. Make every economic signer explicit.
4. Prefer human recovery over irreversible automation in early versions.
5. Treat autonomy as a spectrum, not a binary.
6. Design for machine-readable state and errors.
7. Make invalid authority obvious before transaction submission.
8. Distinguish direct payment use cases from conditional-settlement use cases.
9. Promote research into product only after evidence.

## Governance

This repository uses an RFC process for proposals.

- `Draft`: early idea, open to broad critique.
- `Research`: actively gathering evidence, examples, or experiments.
- `Candidate`: coherent proposal ready for deeper review.
- `Accepted`: useful enough to guide experiments or implementation.
- `Superseded`: replaced by a better proposal.
- `Rejected`: intentionally not pursued, with reasoning preserved.

Accepted RFCs are still research outputs. They do not automatically change Trustless Work's production roadmap.

## Success Criteria

The initiative succeeds if it produces:

- a credible public thesis for agentic escrow;
- a reusable vocabulary for agentic settlement;
- a clear authority-level model;
- practical use-case analysis;
- agent-facing tool/API schema proposals;
- x402 composition patterns;
- reference experiments that answer real questions;
- contributor participation from agent, wallet, payments, and protocol builders.

## North Star Question

What financial infrastructure lets agents transact on behalf of humans, organizations, and other agents without giving them unchecked economic authority?
