# Merged Community Evidence Map - 2026-09-21

Status: Research reference  
Scope: merged community contributions  
Normative status: non-normative; does not rewrite or approve any RFC

## Purpose

This note records the current merged status of the first four community contributions and maps each contribution to the use case, RFC, or research question it now informs.

It is a freshness note for repository interpretation. It does not change RFC status, production roadmap, API design, SDK behavior, MCP behavior, or smart-contract scope.

## Merged Contribution Map

| Contribution | Current repository artifact | Primary research lesson | Informs |
| --- | --- | --- | --- |
| PR #1 - bounded-authority milestone payouts + x402 | [Nirium bounded-authority milestone payouts](../use-cases/nirium-bounded-milestone-payouts.md) | Human defines the economic boundary; automation operates inside that boundary. | [RFC 0002](../rfcs/0002-agent-authority-levels.md), [RFC 0003](../rfcs/0003-x402-escrow-composition.md), [RFC 0004](../rfcs/0004-agent-facing-tool-schema.md) |
| PR #2 - authority by missing parameter | [Nirium treasury missing parameter](../use-cases/nirium-treasury-missing-parameter.md) | Make unauthorized economic actions inexpressible, not merely disallowed. | [RFC 0002](../rfcs/0002-agent-authority-levels.md), authority and delegation research |
| PR #3 - direct x402 payment without escrow | [Nirium direct x402 payment](../use-cases/nirium-direct-x402-payment.md) | Do not introduce conditional settlement when there is no meaningful condition to protect. | [RFC 0003](../rfcs/0003-x402-escrow-composition.md), [RFC 0004](../rfcs/0004-agent-facing-tool-schema.md) |
| PR #4 - production evidence for agent-facing tool schema | [Production evidence for an agent-facing tool schema](../rfcs/0005-agent-facing-tool-schema-production-evidence.md) | Version-specific workarounds are evidence; durable agent interfaces should encode state, authority, signer mode, and recovery semantics. | [RFC 0004](../rfcs/0004-agent-facing-tool-schema.md), research-note process |

## Interpretation Notes

- The contributions have been merged into `main` and integrated into the repository narrative.
- The repository now treats them as implementation evidence, not as accepted production design.
- The `rfcs/0005-...` path is retained for provenance, but repository governance classifies it as a Research Note supporting RFC 0004.
- The contributions should be cited as merged research evidence only after reviewing their specific claims, failure modes, and maintainer caveats.

## Durable Research Distinctions

1. Bounded authority can be created by pre-authorizing the economic object an automated signer may touch.
2. Bounded authority can also be created by removing dangerous parameters from the callable interface.
3. Direct payment can be correct when fulfillment is immediate, low-value, and no conditional release is needed.
4. Counterparty or fulfillment risk is separate from payment-infrastructure or dependency risk.
5. Agent-facing tooling needs explicit semantics for signer mode, caller capability, retry safety, chain submission, and read-model convergence.

## Analysis To Do

- Compare the four merged contributions against the external protocol reference notes.
- Decide which lessons belong in future RFC amendments, if any.
- Keep RFC edits blocked until explicit maintainer approval.
