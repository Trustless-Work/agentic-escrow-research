# Curated learnings

Short, agent-readable summaries of research-supported lessons in this repository.

The machine-readable catalog is [`../learn-index.json`](../learn-index.json). Read that first when discovering lessons programmatically. Read [`../AGENTS.md`](../AGENTS.md) for artifact semantics and confidence labels.

## Current lessons

| ID | Question | Confidence | Evidence |
| --- | --- | --- | --- |
| [settlement/direct-vs-escrow](settlement-choice.md) | When should an agent pay directly versus use escrow? | limited | mainnet + testnet + use cases |
| [authority/bounded-execution](bounded-authority.md) | How should delegated agent authority stay safe? | limited | mainnet patterns + testnet contracts + implementation |
| [recovery/settled-not-delivered](recovery-semantics.md) | What recovery facts must an agent see after payment or escrow actions? | limited | mainnet + testnet + production fixes |

## What these files are not

- Not accepted RFCs
- Not approved Trustless Work API or contract changes
- Not a substitute for the full research notes they cite

When extending this set, update `learn-index.json`, add or revise the Markdown lesson, cite sources with relative links, and state whether the change revises an existing finding.
