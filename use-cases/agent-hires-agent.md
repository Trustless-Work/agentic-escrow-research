# Agent Hires Another Agent

## Summary

One agent hires another agent or autonomous service to complete a task. Payment is escrowed and released when the requested outcome is delivered.

## Actors

- Principal human or organization
- Hiring agent
- Service agent
- Service provider or operator
- Verifier or approver
- Dispute resolver

## Flow

```text
Principal defines objective
Hiring agent scopes task
Service agent accepts terms
Escrow is created
Principal or policy wallet funds escrow
Service agent performs task
Evidence or output is submitted
Verifier approves or disputes
Funds release to provider
```

## Why Direct Payment May Be Insufficient

Agents may promise outcomes that are hard to evaluate upfront. A direct payment creates weak accountability if the downstream agent underperforms.

## Escrow Value

- outcome-based payment;
- evidence-based release;
- role separation between requester, performer, verifier, and release signer;
- audit trail for agent-to-agent commitments.

## Authority Model

The hiring agent should be able to prepare task terms. Funding and approval should remain human or policy-controlled until the use case earns more autonomy.

## Failure Modes

- service agent cannot complete the work;
- output is low quality;
- requester agent gave poor instructions;
- verifier disagrees with service agent;
- task scope changes midstream;
- agents collude or spoof evidence.

## Open Questions

- What counts as completed work?
- Can agent outputs be graded deterministically?
- Should escrow terms include acceptance tests?
- How should revisions be handled?
