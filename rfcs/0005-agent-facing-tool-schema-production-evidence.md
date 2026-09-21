# RFC 0005: Production Evidence for an Agent-Facing Tool Schema

Status: Research

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
