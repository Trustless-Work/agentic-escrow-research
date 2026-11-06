# Agent entry point

Read this file first when connecting an agent to this repository.

This repository is **public research**, not a production protocol, API, SDK, MCP, or smart-contract commitment. A Draft RFC or merged research note is evidence and design discussion. It is not an approved Trustless Work interface.

## Read order

1. **Accepted decisions** in `decisions/` if any exist. None exist yet.
2. **Curated learnings** in [`learn-index.json`](learn-index.json) and [`learnings/`](learnings/). Prefer these over raw history.
3. **Raw research** in [`research-notes/`](research-notes/), [`rfcs/`](rfcs/), and [`use-cases/`](use-cases/) only when a curated lesson points there, or when you need the full evidence trail.

Start discovery with [`learn-index.json`](learn-index.json). Each entry names the lesson, evidence class, confidence, limitations, example files, source notes, and related RFCs.

## Artifact semantics

| Artifact | Meaning |
| --- | --- |
| Curated learning | Short, agent-oriented summary of what the repository currently treats as research-supported. Always carries sources, evidence class, confidence, and limitations. |
| Example JSON | Small illustrative envelope or decision table. Research vocabulary unless an accepted decision says otherwise. |
| Research note | Non-normative evidence, often version-specific. Prefer citing it through a curated learning. |
| RFC | Structured proposal under review. Preserve its original questions; later evidence updates live in dedicated sections. |
| Use case | Concrete scenario or implementation example. |
| Accepted decision | Explicit research or product decision recorded in `decisions/`. Stronger than a learning; still not automatically a production commitment. |

## Why curated learnings exist

Do not duplicate long research-note prose into agent context. Curated files exist so an agent can:

- answer a common question with a short summary;
- see whether the claim is mainnet, testnet, implementation-only, private, or unverifiable;
- follow relative links into the exact source note or use case;
- avoid treating Draft RFCs as approved APIs.

If a curated summary and a source note disagree, trust the source note and treat the curated file as needing an update.

## Confidence and evidence labels

Use the labels in `learn-index.json` and each learning file:

- **Evidence class:** `mainnet`, `testnet`, `implementation`, `source-only`, `private`, `unverifiable`
- **Confidence:** `strong`, `limited`, `weak`
- **Status:** `research-supported`, `open`, `superseded`

Never infer production readiness from testnet success, private-repo fixes, or a single team's throwaway-key experiments.

## Style for contributors

Keep curated learnings short. Prefer:

- one question;
- a few durable lessons;
- an evidence table;
- explicit limitations;
- relative Markdown links to sources;
- pointers to example JSON when useful.

Do not invent approved field names, contract behavior, fee addresses, or supported configurations. Mark research vocabulary as such.
