# Contributing

Thank you for contributing to Agentic Escrow Research.

This repository welcomes builders, researchers, protocol designers, AI-agent developers, wallet teams, payment teams, marketplaces, security reviewers, and curious operators.

## Ways to Contribute

- propose a new use case;
- critique an RFC;
- add a diagram;
- document a failure mode;
- compare direct payment vs escrow;
- design a tool schema;
- build a small reference experiment;
- write threat-model notes;
- improve the whitepaper;
- add a research note that maps implementation evidence to an existing RFC.

## Contribution Standards

Good contributions should:

- separate facts from assumptions;
- explain the trust model;
- name the economic signer;
- describe failure and recovery paths;
- avoid claiming production behavior without evidence;
- prefer minimal primitives over unnecessary contract complexity.

## RFC Process

Create RFCs under `rfcs/` using this naming pattern:

```text
0000-short-title.md
```

Use the following status values:

- `Draft`
- `Research`
- `Candidate`
- `Accepted`
- `Superseded`
- `Rejected`

## Research-Note Process

Use research notes for implementation evidence, operational observations, version comparisons, and evidence that informs an existing RFC without proposing a new normative design.

Research notes should:

- identify which RFC or research question they inform;
- distinguish production, testnet, historical, and unverified evidence;
- preserve version context;
- mark implementation-specific workarounds clearly;
- state whether a newer version addresses, partially addresses, or still needs to validate the observed failure mode.

Index research notes under `research-notes/`. A legacy or contributor-provided path may be retained for provenance, but the repository classification should remain explicit.

## Use-Case Process

Create use cases under `use-cases/` using this structure:

```md
# Use Case Name

## Summary
## Actors
## Flow
## Why Direct Payment May Be Insufficient
## Escrow Value
## Authority Model
## Failure Modes
## Open Questions
```

## Research Discipline

This is a research initiative, not a marketing repository. Strong arguments, clear tradeoffs, and precise uncertainty are more valuable than broad claims.
