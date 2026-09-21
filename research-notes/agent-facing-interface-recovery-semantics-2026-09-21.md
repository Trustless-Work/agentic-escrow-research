# Agent-Facing Interface Recovery Semantics - 2026-09-21

Status: Research reference  
Scope: machine-readable state, signer mode, retry safety, and recovery semantics  
Normative status: non-normative; does not rewrite or approve any RFC

## Purpose

This note keeps the broader recovery-semantics research close to RFC 0004 without changing RFC 0004 itself.

The merged production-evidence note already shows that agents need more than success/failure booleans. This file collects the reference vocabulary we should analyze before proposing any schema changes.

## Candidate State Vocabulary

```text
prepared
unsigned
signed
submitted
confirmed
lagging
failed
```

## Candidate Response Vocabulary

```text
requiredRole
signingMode
authorityScope
transactionState
readModelState
settlementState
retrySafe
dependencyState
nextValidActions
```

These names are research vocabulary, not approved API fields.

## Evidence Inputs

| Source | Relevant lesson |
| --- | --- |
| RFC 0004 | Agent-facing tools need typed, deterministic, state-aware, machine-readable operations |
| RFC 0005 research note | Chain submission and read-model convergence are different states; signer mode changes the correct flow |
| Nirium direct x402 payment | Payment rejection, facilitator outage, ambiguous settlement, retry safety, and reconciliation must be legible to unattended clients |
| x402 / MPP / AgentCore payment docs | Payment infrastructure may expose scheme, session, wallet, or facilitator-level failure states |
| Trustless Work historical MCP | Mutating operations may produce unsigned XDR for human inspection; this is useful evidence but not a future design requirement by itself |

## Open Analysis Questions

- Should tool discovery expose only currently authorized operations, or the complete schema with explicit capability metadata?
- How should an agent distinguish "submitted but not indexed" from "not submitted"?
- What signer metadata is needed before an economic action is prepared?
- Which failures are retryable, terminal, ambiguous, or dependency-related?
- Should direct-payment and escrow-backed flows share a common recovery envelope?

## References

- <https://github.com/Trustless-Work/agentic-escrow-research/blob/main/rfcs/0004-agent-facing-tool-schema.md>
- <https://github.com/Trustless-Work/agentic-escrow-research/blob/main/rfcs/0005-agent-facing-tool-schema-production-evidence.md>
- <https://github.com/Trustless-Work/agentic-escrow-research/blob/main/use-cases/nirium-direct-x402-payment.md>
- <https://github.com/x402-foundation/x402/blob/main/specs/x402-specification-v2.md>
- <https://stripe.com/blog/machine-payments-protocol>
- <https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/payments-process-payment.html>
