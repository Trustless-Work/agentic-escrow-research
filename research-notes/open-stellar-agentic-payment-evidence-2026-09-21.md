# Open Stellar Agentic Payment Evidence - 2026-09-21

Status: Research reference  
Scope: Stellar x402 payments, agent skills, ZK spend-cap authorization, evaluation-driven release reasoning  
Normative status: non-normative; does not rewrite or approve any RFC

## Purpose

This note maps implementation evidence from the Open Stellar project into the Agentic Escrow Research questions around direct payment, bounded agent authority, verification envelopes, and agent-facing tool schemas.

Open Stellar is a Stellar-focused agent infrastructure project. It combines an agent registry, x402-style paid service calls, a skills marketplace, receipts, a ZK Agent Passport concept, Soroban escrow experiments, and an admin console for operating cloud agents.

This is not production Trustless Work evidence. It is useful as adjacent implementation evidence for identifying which parts of an agentic payment workflow are already expressible as direct Stellar payments, which parts need conditional settlement, and which metadata agents need before acting economically.

## Repository Evidence

Open Stellar currently includes these relevant primitives:

| Area | Evidence | Research relevance |
| --- | --- | --- |
| Stellar x402 quotes and receipts | `lib/protocols/x402.ts`, `app/api/protocol/x402/*`, `app/api/explorer/receipts` | Direct per-request payment, quote expiry, receipt registry, chain choice, payer binding |
| Skills marketplace | `docs/features/skills-marketplace.md`, `app/api/agents/[id]/skills/*` | Agent-to-agent service purchase flow where direct payment triggers execution |
| Agent Passport / spend cap | `lib/passport/*`, `components/admin/passport-panel.tsx`, `app/api/protocol/passport/*` | Bounded authority and proof-like authorization before payment settlement |
| Soroban escrow experiment | `contracts/stellar/escrow/src/lib.rs` | Minimal conditional settlement state machine on Stellar/Soroban |
| Cloud agents and JEV evaluation | `lib/agent-runtime/*`, `lib/ai/jev.ts`, `app/api/ai/jev/evaluate` | Typed model evaluation for readiness, risk, and verification decisions |
| CosmosPay integration draft | `lib/cosmospay/client.ts`, `app/api/cosmos/*` | SEP-7-style Stellar payment intents adjacent to x402 receipts |

## Current Open Stellar Flow

The direct service flow is roughly:

```text
consumer/agent discovers a skill
-> requests a 402 quote for skill invocation
-> pays on Stellar or another supported rail
-> submits tx hash and payment reference
-> Open Stellar validates the settlement shape
-> receipt is recorded
-> skill endpoint is invoked
-> result is returned to the paying agent
```

This is a direct-payment pattern, not an escrow pattern. It fits small, immediate, API-like work where the service response is returned in the same flow and there is no meaningful post-payment delivery condition to protect.

The current skills documentation is explicit that a failed endpoint call can still deduct payment. That is an important boundary condition for this research: for synchronous paid API calls, the payment rail may be enough; for promised outcomes, revisions, milestones, or subjective quality, direct payment creates weak recourse.

## Mapping to Existing Research

### RFC 0003: x402 and escrow composition

Open Stellar reinforces the repo's existing distinction between direct payment and conditional settlement.

Direct x402-style payment is a good fit when:

- the service is low-value or metered;
- the paid output is produced immediately;
- a receipt plus retry/reconciliation semantics are sufficient;
- the payer accepts that payment buys an attempt or response, not a milestone outcome.

Escrow becomes useful when:

- the result is delivered later;
- the result is qualitative or reviewable;
- multiple milestones or revisions are expected;
- a third party or verifier decides whether the output satisfies the agreement;
- the paying agent should not be able to release funds unilaterally.

This suggests a composition boundary:

```text
x402 / MPP / CosmosPay intent = pay for access, attempt, usage, or immediate response
Trustless Work escrow = commit funds for an outcome that still needs evidence, approval, dispute, or recovery
```

### RFC 0004: agent-facing tool schema

Open Stellar's quote and settlement flow already exposes concepts an agent-facing schema should preserve:

- `quoteId`
- `paymentRef`
- `serviceId`
- `chain`
- `payer`
- `amountUnits`
- `expiresAt`
- `memo`
- `txHash`
- `receipt`
- `accepted`

For a Trustless Work agent-facing tool, similar fields should be extended with escrow-specific state rather than hidden behind generic success/failure responses.

Candidate response envelope fields informed by Open Stellar and the existing RFC 0004 discussion:

```text
operationKind: direct_payment | escrow_commitment | escrow_release | dispute
paymentRail: stellar_x402 | mpp | sep7_intent | trustless_work_escrow
serviceId
agreementId
milestoneId
payer
receiver
amount
quoteExpiresAt
paymentRef
transactionState
readModelState
receipt
verificationState
passportOrPolicyState
retrySafe
nextValidActions
```

The important design point is that an agent should know whether it is buying access, committing funds, approving evidence, releasing funds, or disputing a result. Those operations have different authority and recovery semantics.

### RFC 0002: authority levels

The ZK Agent Passport concept in Open Stellar is adjacent to bounded authority. It models an agent as authorized only up to a spend cap and uses a passport/attestation gate before settlement.

This supports two research principles already present in the Trustless Work repo:

- spending ability should not imply release authority;
- proof or policy checks should be visible before an economic action is attempted.

Open Stellar's current passport evidence should be treated as experimental and adjacent, not as a production privacy or authorization claim. Its research value is the shape of the control: an agent-facing flow can require a machine-readable precondition before accepting or releasing payment.

### Verification and evidence envelope

The JEV integration draft is relevant because it treats model output as a typed evaluation rather than chat. That maps naturally to research questions such as:

- is the task actionable?
- is the release condition satisfied?
- is the risk low enough for automated continuation?
- does the submitted evidence match the milestone criteria?

For Trustless Work research, a model evaluator should not be confused with a final economic signer. A safer architecture is:

```text
evidence submitted
-> evaluator produces typed verdict + confidence + reasons
-> policy decides whether human review is still required
-> authorized approver/release role signs escrow state transition
```

This fits the current research direction: keep richer verification logic above the audit-bound contract layer until the evidence model is validated.

## Stellar Agentic Ecosystem Context

The Stellar developer docs now describe agentic payments as HTTP-native payment protocols for AI agents and APIs. They identify x402 on Stellar and MPP on Stellar as the primary documented paths for machine-readable per-request payments and direct settlement.

The Stellar Skills directory also lists adjacent agentic and escrow-related projects, including:

- Trustless Work Escrow skill;
- Agent Payments (x402 + MPP) skill;
- Eunomia bounded agent treasury;
- Stellar Agent Search for ERC-8004-style agent discovery;
- Cogladius for on-chain tasks and escrowed judge payouts;
- Nirium Agentic Payments;
- MPP Discover and ROZO Checkout for agent-paid services.

That ecosystem context supports treating Trustless Work escrow as one layer in a larger Stellar agentic payment stack rather than as a replacement for direct payment rails.

## Research Contribution From Open Stellar

Open Stellar can contribute an end-to-end reference scenario:

```text
Agent A discovers Agent B's skill
Agent A pays directly through Stellar x402 for a cheap immediate call
Agent B returns a result and a receipt is recorded
If the task is larger or milestone-based, Agent A prepares an escrow instead
Evidence is collected per milestone
A typed evaluator recommends approve/dispute
A human or policy approver performs the release action
```

This scenario can test the boundary between:

- paid API/service access;
- paid attempt;
- paid outcome;
- milestone escrow;
- evaluator-assisted approval;
- final signer authority.

## Failure Modes Observed or Implied

- A skill endpoint may fail after direct payment. This is acceptable only if the product definition says payment buys an attempt or immediate API response.
- Quote expiry and payment submission are separate states. Agents need to know whether retrying creates a duplicate economic action.
- A tx hash format check is not the same as full settlement verification. Production flows need chain verification, read-model convergence, and reconciliation semantics.
- Multi-chain payment options can confuse authority and settlement semantics unless the quote binds payer, chain, destination, amount, memo, and expiry clearly.
- A model evaluator such as JEV can support release reasoning but should not silently become the economic signer.
- User-supplied AI Gateway keys create a BYOK boundary that should be modeled separately from protocol authority.

## Open Questions

- Should an agent-facing Trustless Work tool expose a `paymentPurpose` field distinguishing access, attempt, usage, commitment, release, refund, and dispute?
- Can the same discovery surface return both direct-payment services and escrow-required services, with machine-readable settlement requirements?
- What minimum receipt fields are needed for a paid agent skill to become evidence in a later escrow dispute?
- Should a verifier/evaluator output be hashable and attachable to escrow evidence without being stored on-chain?
- How should Trustless Work tools express that an evaluator recommended release but the caller still lacks release authority?
- Can Stellar x402/MPP receipts and Trustless Work escrow state share a common evidence envelope for agent-to-agent commerce?

## Maintainer / Reviewer Flags

- Treat this as adjacent implementation evidence, not a production recommendation.
- Separate Open Stellar's current mocked or local-only flows from deployed, audited, or testnet-verified claims.
- Do not infer that every skill invocation should use escrow. The strongest contribution is the boundary between direct paid calls and outcome-conditioned settlement.
- Do not infer that JEV or any model evaluator should be a release signer. The safer pattern is evaluator-as-evidence or evaluator-as-policy-input.

## References

- Open Stellar repository: <https://github.com/Bitcoindefi/Open-Stellar>
- Stellar Agentic Payments docs: <https://developers.stellar.org/docs/build/agentic-payments>
- Stellar Skills directory: <https://skills.stellar.org/>
- Trustless Work RFC 0003: `rfcs/0003-x402-escrow-composition.md`
- Trustless Work RFC 0004: `rfcs/0004-agent-facing-tool-schema.md`
- Trustless Work verification note: `research-notes/verification-and-evidence-envelope-2026-09-21.md`