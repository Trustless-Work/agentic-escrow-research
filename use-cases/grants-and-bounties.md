# Grants and Bounties

## Summary

Agents help match funding to work, track milestones, collect evidence, and recommend release for grants or bounties.

## Actors

- Grant funder or bounty sponsor
- Applicant or contributor
- Research or matching agent
- Milestone reviewer
- Release signer
- Dispute resolver

## Flow

```text
Sponsor defines funding objective
Agent helps match applicant or bounty worker
Milestones are defined
Escrow is created and funded
Contributor submits work
Evidence is reviewed
Milestone is approved or disputed
Funds release
```

## Why Direct Payment May Be Insufficient

Grants and bounties often pay for future work. Direct upfront payment weakens accountability, while pay-after-delivery may reduce contributor confidence.

## Escrow Value

- credible funding commitment;
- milestone-based release;
- transparent status;
- evidence trail;
- reviewer role separation;
- reusable capital allocation primitive.

## Authority Model

Agents can help discover applicants, structure milestones, monitor evidence, and recommend actions. Funding and release should be controlled by the sponsor, reviewer, or defined policy.

## Failure Modes

- contributor does not deliver;
- milestones are vague;
- reviewer bias;
- duplicate claims;
- weak evidence;
- sponsor abandons funded escrow;
- agent recommends poor matches.

## Open Questions

- What evidence should be required per milestone type?
- Can reviewer decisions be made more transparent?
- Should agents help identify likely fraud or low-quality submissions?
- How should abandoned escrows be resolved?
