# RFC 0005: Production Evidence for an Agent-Facing Tool Schema

Status: Research

Maintainer classification: **Research Note supporting RFC 0004 — non-normative**  
Version context: **V1 is the current mainnet production baseline; V2 is beta/testnet.**  
Provenance note: the original RFC-style title/path is retained so the contributor's evidence is not deleted or rewritten away. For repository interpretation, this document is indexed and cited as implementation/research evidence for RFC 0004 rather than as a new normative RFC.

> **Maintainer interpretation rule:** preserve the V1 production observations as evidence, then map each observation to V2 as **addressed**, **partially addressed**, **still open**, or **requires beta validation**. A V2 improvement does not invalidate the V1 observation; it tells us what the newer architecture learned from it.


## Summary

RFC 0004 sketches candidate operations, a transaction state model, and an
error model for an agent-facing escrow interface, and asks four open
questions about how it should be designed. This RFC doesn't propose a new
schema — it contributes real operational evidence from a shipped tool
surface that already covers part of RFC 0004's shape, plus two production
failure modes (documented in this repo's PR #1) that bear directly on the
state model and error model sections.

Everything cited here is either independently verifiable on Horizon or a
live mainnet API today, or explicitly marked as not yet verified — same
evidence discipline as PR #1/#2.

## Maintainer V1 → V2 Mapping

The observations below are valuable precisely because they come from the production V1 generation. V2 should be evaluated against them rather than used to dismiss them.

| Observation in this note | V1 production interpretation | V2 beta/testnet mapping | Research status |
| --- | --- | --- | --- |
| Chain submission succeeds while the indexed/read model lags | Real operational failure mode; V1 recovery could involve direct Horizon submission and manual `update-from-txhash` reconciliation | V2 explicitly models this through `POST /stellar/send-transaction` and the `STELLAR_TX_SUBMITTED_INDEXER_LAGGING` result. Current guidance treats this as successful submission with eventual read-model consistency and says **do not retry the transaction** | **Addressed in V2 design; retain as regression evidence** |
| A deploy bypasses the API and the backend never learns that the object exists | V1 can end up with valid chain state that application state cannot resolve normally | V2 architecture uses continuous event/indexer discovery of Trustless Work escrows and separates chain authority from the read model. This is intended to reduce the failure mode, but the exact bypass/deploy scenario should be tested end-to-end before calling it resolved | **Architecturally improved; beta validation required** |
| `getEscrowByContractId` receives the wrong input shape and silently returns no result | V1 caller-side validation can be too weak; absence of an error is not proof of absence of state | V2 surfaces use typed `contractIds` filters and stricter declared-field/type validation. That improves the interface contract, but the original silent-failure shape should remain a regression test | **Partially addressed; regression test required** |
| Human signing can miss a tight transaction-validity window | Real friction in a prepare → human sign → submit workflow | V2 still exposes an unsigned-XDR → signer → `send-transaction` workflow. No evidence in this review proves that the timing constraint is eliminated | **Still open; test by signer type** |
| Tool schema and public availability differ | Real-world tools can exist internally or for restricted signers without being public to arbitrary third parties | V2 does not remove this distinction; capability exposure remains a policy/product decision | **Still relevant** |
| Human and automated signers need different interaction shapes | Human signers benefit from inspectable unsigned transactions; constrained automated signers may benefit from bounded complete actions | V2 still makes signer workflow an explicit integration concern; policy-controlled complete-action semantics remain research work | **Still open / design input** |

### Version warning on the V1 recovery path

The contributor's `Horizon -> update-from-txhash` recovery sequence below should be read as **historical V1 operational evidence**, not as the recommended V2 integration path.

For V2 beta/testnet, the current intended pattern is conceptually:

```text
build action
-> receive unsigned XDR
-> authorized signer signs
-> POST /stellar/send-transaction
-> confirmed
   or STELLAR_TX_SUBMITTED_INDEXER_LAGGING
-> re-read / inspect transaction state
```

The important research lesson survives across versions:

> **Chain submission state and application/read-model state are separate machine states and must not be collapsed into one success/failure boolean.**

## What's Shipped

Three agent-facing surfaces predate RFC 0004 but map onto its
candidate-operations shape:

- An **MCP read-only tool surface** — `get_nodes`, `get_reporting_summary`,
  `get_treasury_info` / `vault` / `vaults` / `strategy_asset`,
  `agent.getMarket()`. These only read state, never execute a mutating
  action — confirmed live today: `GET /api/reporting/summary` → `200`,
  `GET /api/audit/info` → `200`, `GET /api/v1/premium/market` → `402`
  (a genuine, payable x402 challenge, not a stub).
- A **CLI** (`nirium doctor`, `nirium verify`) — same read-only shape,
  published on npm.
- A **published Claude Skill**, `nirium-agentic-payments`, live on
  skills.stellar.org — an agent-facing tool definition for the
  discover→pay→settle x402 flow, mainnet-verified (settlement
  `3134a51c66091fd7fbd85b38a4a6ec6cd432bb92c2450eac84ea7855cb7558bc`,
  cited in PR #3 of this repo).

None of these currently implement RFC 0004's mutating operations
(`fund_escrow`, `approve_fulfillment`, `release_payment`, etc.) as a
public, third-party-callable surface — the escrow-mutating signers (the
allowlist-scoped Payouts signer in PR #1, the no-destination Treasury
signer in PR #2) exist and are tested on testnet and mainnet, but haven't
been opened to third-party callers. Naming this precisely: the evidence
below speaks to tool *design*, not to "here is a public API you can call."

## Candidate Operations — What Maps, What's Missing

| RFC 0004 operation | Nirium equivalent | Public today? |
| --- | --- | --- |
| `get_escrow` | `get_treasury_info`/`vault` (read escrow-adjacent state) | Yes, read-only |
| `create_protected_payment` | none directly; the `market` endpoint's own `402` challenge is the closest direct-payment analog (no escrow creation) | Partial — direct payment only |
| `fund_escrow` | internal, tested on testnet + a self-directed mainnet cycle | No |
| `submit_fulfillment` | none built | No |
| `approve_fulfillment` / `release_payment` | `approve-milestone`/`release-milestone` (PR #1) | No |
| `raise_dispute` | Dispute Resolver role exists in the design (PR #1); no tool wraps it | No |
| `cancel_or_refund_if_permitted` | not built | No |

The gap is informative on its own: everything made public so far is
read-only or single-shot direct payment; everything that mutates escrow
state has stayed behind an invite gate for reasons unrelated to the tool
design itself. Worth naming as a real-world pattern RFC 0004 might want
to account for: a tool schema's *design* and its *public availability*
can be decoupled for reasons that have nothing to do with whether the
schema is good.

## State Model — Real Evidence for `lagging`

RFC 0004 already lists `lagging` ("chain submission appears successful
but indexer has not caught up") as a state. This isn't hypothetical here
— it's the exact failure documented in PR #1: a mainnet escrow deploy
submitted directly to Horizon succeeded on-chain
(`4bb8fefa0667a45fc61bd5ec172b5e83b095d579c2b65728494bbc832604eb98`,
2026-09-02) while the escrow protocol's own backend had no record of it —
`fund-escrow` returned "Escrow not found" against a contract that was
genuinely live. The fix that worked (documented in PR #1): submit signed
XDR directly to Horizon, then call the protocol's own
`update-from-txhash` indexer endpoint to register the settlement after
the fact — a real, working recovery mechanism, but only for the
*settlement* of an already-registered escrow, not for a deploy that
bypassed the API entirely. That distinction — `lagging` covering
settlement-after-deploy vs. an unrecoverable "the backend never learned
this object exists" case — isn't visible in RFC 0004's current state
table, and may be worth a fifth state or an explicit note that `lagging`
doesn't uniformly resolve on its own.

### Maintainer flag: do not generalize the V1 workaround

The operational incident is valid V1 evidence. The recovery mechanism is version-specific.

- **Keep:** the observation that chain-confirmed state can temporarily disagree with the application read model.
- **Keep:** the requirement for a machine-readable state that tells an agent not to duplicate an already-submitted economic action.
- **Flag as V1-specific:** manual `update-from-txhash` reconciliation.
- **Map to V2:** `STELLAR_TX_SUBMITTED_INDEXER_LAGGING` represents successful submission with delayed read-model convergence.
- **Validate on V2 beta:** whether an API-bypassing deploy is always rediscovered automatically by the continuous indexer path.

Do not infer from this note that V2 clients should reproduce the V1 direct-Horizon/manual-indexer flow.

## Error Model — Two Real, Named Failures

Both already reported and fixed, both directly relevant to RFC 0004's
"what precondition failed" / "is retry safe" questions:

- `getEscrowByContractId` silently fails on a bare string instead of the
  documented `contractIds[]` array — no error surfaced, just an empty
  result. A tool wrapping this call needs to validate its own inputs
  rather than trust the absence of an error.
- The tight signing window measured in PR #1 (well under two minutes,
  the fastest tested human path still arrived late): a tool that
  separates "prepare transaction" from "sign" as two calls, the way RFC
  0004's `prepared`/`unsigned`/`signed` states imply, needs to either
  collapse those steps for automated signers or expose the window's
  length so a caller can judge whether manual signing is viable at all.

### Maintainer flag: validation and signing-window status

The two failures above map differently into V2:

- The malformed `contractIds` case appears **partially addressed** by typed filters and stricter declared-field/type validation in V2-facing interfaces. Keep the original incident as a regression test rather than assuming the class of bug is impossible.
- The signing-window observation is **not confirmed solved** by V2. V2 still has a prepare/unsigned/sign/submit lifecycle for human-controlled signing. The research question therefore remains signer-specific: human review may justify a multi-step flow; a narrowly scoped automated signer may need a bounded atomic action to avoid timing and handoff failures.

This distinction should feed RFC 0004 without forcing one signing mode on every caller.

## Additional Trustless Work Evidence: Historical MCP Surface

The contributor's Nirium evidence should be compared with Trustless Work's own earlier MCP implementation rather than treated as the only available implementation data point.

The historical `Trustless-Work/trustlesswork-mcp` repository predates this research initiative and exposes tools including:

- `deploy_single_release_escrow`
- `deploy_multi_release_escrow`
- `fund_escrow`
- `update_escrow`
- `approve_milestone`
- `change_milestone_status`
- `release_funds`
- `release_milestone_funds`
- dispute and dispute-resolution operations
- escrow query operations
- `send_transaction`
- `withdraw_remaining_funds`

Most mutating operations return unsigned XDR for a separate signer, so this implementation is useful historical evidence for the **inspectable human-signing** side of RFC 0004.

It is explicitly **V1-oriented** and mirrors many backend/API concepts fairly directly. It should not be treated as the future agent-facing design. Instead, the stronger evidence set is:

```text
historical Trustless Work MCP
        +
Nirium operational evidence
        +
current Core API V2 beta architecture
        ↓
agent-facing design principles for RFC 0004
```

This comparison supports a key research direction: the future tool layer should model **economic intent, authority, signer mode, state and recovery semantics**, rather than mechanically exposing whichever backend endpoints happen to exist in one implementation generation.

## Answers to RFC 0004's Open Questions, From Operational Experience

- **"Should these tools map one-to-one to Core API actions?"** Experience
  here argues no, at least not naively: the path that worked (PR #1) was
  direct-to-Horizon submission plus a separate registration call, not a
  single call to the protocol's own send-transaction helper. A 1:1 tool
  wrapping only the latter would have inherited the tighter window.
- **"Should tool calls generate unsigned transactions or complete
  actions?"** For an automated, scope-restricted signer (both patterns in
  PR #1/#2), collapsing to complete actions removed the actual source of
  delay — manual XDR handling — not the network route itself. This
  suggests the answer may depend on signer type: human-in-the-loop wants
  unsigned transactions to review; automated policy signers want complete
  actions so a multi-step handoff isn't the bottleneck.
- **"How should policy wallets and human approval fit into the schema?"**
  Both patterns here split this at the *object* level, not the *call*
  level: a human-signed creation step fixes every destination address up
  front, with no time pressure, and a separate, narrow-scoped automated
  signer is only ever allowed to act on an object a human already
  created — never to name a new destination. Worth considering as a
  schema-level constraint rather than leaving policy-vs-automation
  entirely to caller discretion.
- **"Should agents receive higher-level commerce primitives instead of
  raw escrow functions?"** No direct evidence either way from this build
  — flagged as genuinely open, not answered here.

## Maintainer Synthesis for RFC 0004

Taken together, the contributor evidence plus the V1/V2 comparison supports the following non-normative design inputs for RFC 0004:

1. **Interface intent should sit above transport details.** An operation such as `release_payment` should not require an agent to reason about internal endpoint topology.
2. **Signer mode is part of the tool contract.** A human-controlled signer and a policy-constrained automated signer may legitimately receive different execution flows.
3. **Authority and capability discovery are separate from operation naming.** A schema may define an operation without exposing it to every caller.
4. **Chain state and read-model state must be independently legible.** “Submitted,” “confirmed,” and “indexed/queryable” are not synonyms.
5. **Retry safety must be explicit.** A temporary read-model lag must never cause an agent to duplicate a successful economic action.
6. **Input contracts should fail loudly and structurally.** Empty results caused by malformed inputs are dangerous for autonomous callers.
7. **Version-specific workarounds are evidence, not interface requirements.** V1 incidents should become V2 regression tests and design lessons rather than permanent public abstractions.

A future agent-facing response envelope may need fields conceptually similar to:

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

These names are research vocabulary, not approved API fields.

## Failure Modes / Scope Honesty

- This is evidence from one integrator's build, not a claim that this
  shape is the right one — worth weighing as one real data point, not a
  reference implementation to copy.
- The mutating half of the tool surface (fund/approve/release) is real
  and tested (PR #1/#2) but not public — a reader can't call it today to
  verify independently. Only the read-only/direct-payment half above is
  independently checkable right now (endpoints cited return live
  `200`/`402` as of this writing).
- No claim is made about whether the object-level authority-boundary
  split generalizes past the milestone-payout and treasury-rebalance
  shapes already documented in PR #1/#2.

## Open Questions

- Does the `lagging` state need to distinguish "settlement not yet
  indexed" from "the backend never learned this object exists," given
  that only the first recovers via the documented indexer mechanism?
- Should RFC 0004 require tool schemas to declare which mutating
  operations are actually callable by a given caller class (public,
  invite-gated, self-directed-only), since the same operation can exist
  in code while being intentionally unavailable for reasons the schema
  itself doesn't capture?

### Maintainer-added V1 → V2 research questions

- Which V1 production failure modes are explicitly covered by V2 regression tests today?
- Can V2 prove that an on-chain deploy missed by the normal submit path is eventually discovered without a manual registration endpoint?
- What transaction-validity window does V2 expose to human signers, and is it machine-readable before signing?
- Should tool discovery return only operations the current caller can actually execute, or return the full schema plus explicit capability metadata?
- What common response envelope can represent chain state, read-model state, signer requirements, retry safety and next valid actions across V1 production and V2 beta?
