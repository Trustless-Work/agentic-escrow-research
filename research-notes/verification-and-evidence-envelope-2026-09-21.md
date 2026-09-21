# Verification and Evidence Envelope - 2026-09-21

Status: Research reference  
Scope: evidence, verification, attestation, and release reasoning  
Normative status: non-normative; does not rewrite or approve any RFC

## Purpose

This note collects research inputs around the central question for outcome-conditioned settlement:

> Why should the money release?

Trustless Work currently records approval through an authorized address. Richer evidence semantics can remain above the stable contract layer while the research validates what agents, platforms, and evaluators need.

## Working Research Object

Candidate off-chain or orchestration-layer object:

```text
VerificationEnvelope {
  agreementId
  milestoneId
  verifier
  verifierType
  criteria
  evidence[]
  proofHash
  signature
  verdict
  timestamp
}
```

This is research vocabulary, not an approved schema.

## Reference Map

| Source | Verification concept |
| --- | --- |
| ERC-8183 | Evaluator can complete or reject; optional reason/attestation hash supports audit and composition |
| VCAP | Independent verification, proof hashes, verifier identity, action logs, release/refund flow |
| ASP | Fulfillment-verification ladder and delayed settlement logic |
| Trustless Work | Authorized Approver signs approval; richer evidence can be held off-chain or in platform state |
| Nirium direct x402 evidence | Even non-escrow flows need receipts, reconciliation, dependency state, and settlement clarity |

## Design Boundary

Keep experimental verification logic outside audit-bound escrow contracts until validated.

Possible placement:

```text
Agent / verifier
-> evidence collection
-> verification envelope
-> policy decision
-> approve or dispute through existing role
```

## Open Analysis Questions

- What evidence is required for digital delivery, compute, bounty work, enterprise procurement, and physical goods?
- Should evidence be anchored by hash, signature, URI, attestation, or a combination?
- Which verifier identities matter: human, platform, agent, oracle, test suite, model evaluator, third-party service?
- What is the minimum evidence envelope that helps agents without requiring a contract change?
- How should a failed or ambiguous verification map to RFC 0004 next actions?

## References

- <https://eips.ethereum.org/EIPS/eip-8183>
- <https://datatracker.ietf.org/doc/html/draft-stone-vcap-02>
- <https://arxiv.org/abs/2609.02208>
- <https://github.com/Trustless-Work/agentic-escrow-research/blob/main/rfcs/0004-agent-facing-tool-schema.md>
- <https://github.com/Trustless-Work/agentic-escrow-research/blob/main/use-cases/nirium-direct-x402-payment.md>
