# RFC 0004: Agent-Facing Tool Schema

Status: Draft

## Summary

This RFC proposes early product-level operations for an agent-facing escrow interface.

The names are conceptual. They are not final API names.

## Design Requirements

Agent-facing tools should be:

- typed;
- deterministic;
- idempotent where possible;
- explicit about required signer and role;
- explicit about transaction state;
- state-aware;
- machine-readable on errors;
- easy to inspect before an economic action;
- strict about invalid authority.

## Candidate Operations

| Operation | Purpose |
| --- | --- |
| `create_protected_payment` | Create or prepare an escrow-backed payment |
| `get_escrow` | Inspect current escrow state |
| `fund_escrow` | Fund a created escrow |
| `submit_fulfillment` | Attach status, evidence, or delivery proof |
| `approve_fulfillment` | Approve completion under the correct role |
| `raise_dispute` | Contest fulfillment or payment release |
| `release_payment` | Release funds when authorized |
| `cancel_or_refund_if_permitted` | Recover or unwind funds when rules allow |

## Transaction State Model

Mutating operations should distinguish:

| State | Meaning |
| --- | --- |
| `prepared` | Tool has built a proposed action |
| `unsigned` | Unsigned transaction exists |
| `signed` | Required signer approved locally or by policy |
| `submitted` | Transaction was sent to the network |
| `confirmed` | Chain and read model reflect the action |
| `lagging` | Chain submission appears successful but indexer has not caught up |
| `failed` | Submission failed or preconditions were invalid |

## Error Model

Errors should tell an agent:

- what precondition failed;
- which role is required;
- whether signing is needed;
- whether the state changed;
- whether retry is safe;
- what next action is valid;
- whether a failure is retryable, terminal, or requires reconciliation;
- whether settlement is confirmed, absent, or ambiguous;
- whether an external dependency is unavailable.

## Direct-Payment Recovery Semantics

The [Nirium direct x402 payment evidence](../use-cases/nirium-direct-x402-payment.md) shows that agent-facing semantics matter even when escrow is not used. An unattended agent must be able to distinguish payment rejection from facilitator unavailability, ambiguous settlement, and a safe retry.

A generic recovery envelope worth researching is:

```text
status
reasonCode
settlementState
retrySafe
dependencyState
nextValidActions
```

This does not imply Trustless Work should standardize x402 errors. It reinforces the broader requirement that agent-facing financial tools expose deterministic recovery semantics rather than opaque transport failures.

## Implementation Evidence: V1 Production → V2 Beta

The [agent-facing tool-schema research note](../rfcs/0005-agent-facing-tool-schema-production-evidence.md) maps Nirium operational evidence and Trustless Work's historical MCP implementation against the current V2 beta/testnet architecture.

The evidence supports several design requirements without prescribing a final API:

- **Version-specific recovery mechanics should not leak into the durable agent abstraction.** V1's direct-Horizon/manual-indexer recovery is evidence for a state problem, not a future tool requirement.
- **Chain state and read-model state are distinct.** V2's `STELLAR_TX_SUBMITTED_INDEXER_LAGGING` is a concrete example: the economic action may already be submitted successfully even while application reads lag.
- **Signer mode matters.** Human-controlled signers may need inspectable unsigned transactions; tightly policy-constrained automated signers may need bounded complete-action flows.
- **Operation definition and caller availability are distinct.** A schema can define `release_payment` without exposing that operation to every caller.
- **Malformed inputs should fail structurally.** Autonomous clients should not interpret an empty result caused by invalid input as proof that an economic object does not exist.
- **V1 incidents should become V2 regression scenarios.** A newer architecture should be evaluated against observed production failures rather than using version change alone as evidence of resolution.

A richer machine-facing result may therefore need to communicate concepts such as:

```text
requiredRole
signingMode
authorityScope
transactionState
readModelState
retrySafe
dependencyState
nextValidActions
```

These fields are research vocabulary, not approved schema names.

## Open Questions

- Should these tools map one-to-one to Core API actions?
- Should agents receive higher-level commerce primitives instead of raw escrow functions?
- Should tool calls generate unsigned transactions or complete actions?
- How should policy wallets and human approval prompts fit into the schema?
- Should the agent-facing layer use a common recovery/error model across direct-payment and escrow-backed actions?
- Should tool discovery expose only caller-authorized operations, or a complete schema plus explicit capability metadata?
- How should V1 production incidents be represented as V2 regression requirements?
- What signer-mode metadata is necessary before an agent prepares an economic action?
