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
- improve the whitepaper.

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
