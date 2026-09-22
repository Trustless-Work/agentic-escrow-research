# Open Stellar Skills Marketplace With Direct Payment and Escrow Boundary

## Summary

Open Stellar models a marketplace where agents register paid skills and other agents or users invoke those skills after an x402-style payment. This use case is useful for Agentic Escrow Research because it shows a clear boundary between direct per-call payment and outcome-conditioned escrow.

A direct payment is appropriate for cheap, immediate, API-like skill calls. Escrow is more appropriate when a skill promises a future outcome, a milestone, a revision cycle, or a result that must be independently verified.

## Actors

- Skill provider agent
- Skill consumer agent
- Principal human or organization behind the consumer agent
- Open Stellar marketplace / registry
- Payment rail or facilitator
- Optional verifier or evaluator
- Optional escrow approver / release signer
- Optional dispute resolver

## Flow

### Direct paid skill call

```text
Provider agent registers a skill with price and endpoint
Consumer agent discovers the skill
Consumer requests a 402 payment challenge
Consumer pays on Stellar or another supported rail
Consumer submits payment proof and paymentRef
Marketplace records a receipt if accepted by the current verification path
Provider skill endpoint is invoked
Consumer receives the response
```

### Escrow-backed skill outcome

```text
Provider agent lists a larger outcome-based service
Consumer agent scopes objective and acceptance criteria
Escrow is created and funded by the principal or policy wallet
Provider performs the work
Evidence is submitted to the marketplace or verifier
Evaluator or reviewer recommends approve, revise, dispute, or refund
Authorized approver/release signer acts on the escrow
Receipt and evidence are retained for audit
```

## Why Direct Payment May Be Insufficient

Direct payment is weak when payment is for an outcome rather than access. If the provider fails, returns low-quality work, or disputes the scope, the consumer has already paid.

Open Stellar's skills documentation explicitly notes that failed endpoint calls may still deduct payment. That is acceptable for some API-like products, but it becomes problematic for bounty work, agent-to-agent tasks, research jobs, code changes, procurement, and deliverables that need review.

## Escrow Value

Escrow adds value when the marketplace needs:

- credible funding commitment before work begins;
- milestone or revision states;
- evidence collection;
- independent review;
- dispute or refund paths;
- role separation between requester, performer, verifier, and release signer;
- machine-readable next actions for agents.

## Authority Model

The consumer agent should be allowed to discover skills, request quotes, compare settlement modes, prepare an escrow draft, and submit evidence.

Early implementations should keep these authorities separate:

| Action | Suggested authority |
| --- | --- |
| Discover skill | Agent |
| Request direct-payment quote | Agent |
| Pay for low-value immediate call | Human-approved wallet or bounded policy wallet |
| Create outcome escrow | Agent prepares, human or policy wallet signs |
| Submit evidence | Provider agent or marketplace |
| Evaluate evidence | Human, verifier, oracle, test suite, or typed evaluator |
| Release funds | Authorized approver/release signer |
| Dispute/refund | Human or policy-controlled role |

## Direct Payment vs Escrow Decision

| Product shape | Suggested settlement |
| --- | --- |
| One API response | Direct x402 or MPP |
| Metered usage | x402 `upto`, MPP session, or subscription rail |
| Cheap deterministic skill | Direct payment with receipt |
| Failed call still counts as billable attempt | Direct payment only if disclosed upfront |
| Multi-step deliverable | Escrow |
| Subjective quality or acceptance criteria | Escrow plus verification envelope |
| Bounty or grant milestone | Escrow |
| Agent hires agent for a task | Escrow when the task has deliverable risk |

## Evidence Envelope

A marketplace can turn direct-payment and escrow flows into a common evidence trail:

```text
SkillEvidence {
  serviceId
  providerAgentId
  consumerAgentId
  paymentRef
  receiptId
  txHash
  chain
  requestHash
  responseHash
  acceptanceCriteria
  evaluator
  verdict
  timestamp
}
```

For direct calls, this envelope supports receipts, retries, audits, and reputation.

For escrow-backed work, the same envelope can support milestone approval, disputes, and release reasoning.

## Failure Modes

- Provider endpoint fails after payment.
- Consumer pays the wrong quote or submits after expiry.
- The marketplace accepts a tx hash shape without full settlement verification.
- A provider returns a syntactically valid but low-quality answer.
- Agents disagree about whether the task was completed.
- Evaluator produces a weak or biased verdict.
- A model evaluator is mistakenly treated as an authorized release signer.
- A paid direct call should have been escrow-backed because the product promised an outcome.

## Open Questions

- Should provider listings declare `settlementMode: direct | escrow_required | hybrid`?
- Should direct paid skill receipts be admissible as evidence in a later escrow dispute?
- How should a marketplace represent that payment bought an attempt rather than successful completion?
- Can a typed evaluator reduce review load without receiving release authority?
- What minimum evidence should be attached to an agent-to-agent task before funds can release?
- Should the marketplace expose a single discovery API for both paid calls and escrow-backed work?

## Research Notes

This use case complements:

- `use-cases/agent-hires-agent.md`
- `use-cases/grants-and-bounties.md`
- `rfcs/0003-x402-escrow-composition.md`
- `rfcs/0004-agent-facing-tool-schema.md`
- `research-notes/open-stellar-agentic-payment-evidence-2026-09-21.md`
