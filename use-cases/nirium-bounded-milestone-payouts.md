# Bounded-Authority Milestone Payouts, Paired With a Direct Access Fee

## Summary

Nirium runs two independent, production-tested primitives that map directly
onto this RFC's composition question:

- **`x402Serve()`**, a direct-payment gate for pay-per-action access — a
  caller gets a `402` with exact terms, signs a Soroban authorization for
  the requested amount, retries, and is granted access once the
  facilitator confirms settlement. No escrow, no waiting, atomic.
- **A Trustless Work Multi-Release Payouts integration**, where a single
  escrow holds several milestones, each with its own receiver, and a
  deliberately narrow-scoped signer is the only thing allowed to approve
  or release an *already-created* milestone — it cannot create a new
  escrow or redirect a payout to an address a human didn't already set.

Both have been run for real, with real funds, and independently verified
against Horizon and Trustless Work's own escrow records — not proposed,
not simulated. This document describes the use case each one covers, the
authority model that keeps the automated half narrow, and the concrete
failure modes found while building it — then asks the open question this
RFC exists to answer: whether the two should compose into one flow.

## Actors

| Actor | Role |
| --- | --- |
| **Requester** | Wants a milestone-based task completed (e.g. a bounty, a multi-part contribution) |
| **Contributor(s)** | One or more parties who complete individual milestones — the receiver differs per milestone, not a single payee for the whole job |
| **Human Operator** | Creates the escrow: sets milestones, amounts, and each receiver. Full authority, no time pressure. |
| **Narrow-Scoped Signer** | A bounded, allowlist-gated automated signer. Can call `approve-milestone` / `release-milestone` on an escrow that already exists. Cannot create escrows, cannot set or change a receiver. |
| **Dispute Resolver** | A distinct, human-held role. If a milestone is contested, only this role can resolve it — the narrow-scoped signer has no authority here at all. |
| **Trustless Work Protocol** | The escrow contract + API + Indexer (see Failure Modes for why the Indexer specifically matters) |
| **x402 Facilitator** | Settles the direct-payment leg for any access fee charged outside the escrow |

## Flow

```text
Requester wants milestone work done
  -> (optional) small x402 fee to access a quote / reserve a work slot
  -> Human Operator creates the escrow: N milestones, N receivers, amounts
  -> escrow funded
  -> Contributor for milestone i completes work, submits evidence
  -> Narrow-Scoped Signer calls approve-milestone(i)
     [blocked if the contract address isn't on its allowlist]
  -> Narrow-Scoped Signer calls release-milestone(i)
     [funds move to milestone i's receiver, and only that receiver]
  -> repeat per milestone
  -> any contested milestone routes to the Dispute Resolver instead of
     the automated path
```

The two payment primitives compose at the edges, not inside the escrow
itself: `x402Serve()` can gate the *cheap, immediate* part (reserving a
slot, buying a quote, paying a small listing fee) while escrow protects
the *larger, delayed, quality-uncertain* part (the actual milestone
payout). Nirium has not yet wired these into a single end-to-end flow —
see Open Questions.

## Why Direct Payment May Be Insufficient

A milestone-based job is exactly the shape RFC 0003 names as needing
escrow, not direct payment: fulfillment happens later than commitment,
quality can't be verified at payment time, and different milestones have
different receivers — a single `x402` payment has no way to express "pay
$X to address A only after condition 1, and $Y to address B only after
condition 2." Collapsing that into one direct payment either forces
prepayment with no recourse, or forces the requester to hold funds and
manually disburse them per milestone — reintroducing exactly the
custodial, ad-hoc trust problem escrow exists to remove.

## Escrow Value

- **Deterministic state per milestone** — each milestone's status
  (pending / approved / released / disputed) is a real, queryable
  on-chain fact, not something either party has to just claim.
- **Receiver correctness is enforced by the protocol, not by the
  automated signer's good behavior.** On Trustless Work's Multi-Release
  contract, the fee destination for `release_milestone_funds` is a
  hardcoded string literal inside the contract itself, not a
  caller-supplied parameter — confirmed by reading the deployed
  `multi-release-main` contract source directly, not assumed from a
  changelog. (A caller-controlled fee-destination bug does exist on the
  *Single-Release* branch of the same repo — a different, separate
  contract Nirium's Payouts integration does not use. Worth naming here
  precisely because this RFC's own contribution standard asks
  contributors to name the economic signer and avoid claiming production
  behavior without evidence — a contract variant one branch over is not
  the same guarantee.)
- **A real dispute path exists and is out of the automated signer's
  reach entirely** — the Dispute Resolver role signs an arbitrary list
  of addresses and amounts to resolve a contested milestone, final and
  immutable once resolved. The narrow-scoped signer has no call it can
  make that touches this path.

## Authority Model

The design splits the flow into two authority tiers instead of trying to
make one signer safe for everything:

1. **Escrow creation is always human-signed, with no time pressure.**
   This is where every receiver address, every amount, and the milestone
   count get fixed. Once created, none of that can be changed by the
   automated half.
2. **Approving and releasing an already-created milestone is what the
   narrow-scoped signer handles.** Because the destinations were already
   fixed in step 1, this signer's blast radius is bounded to "release
   funds to a receiver a human already named" — never "choose who gets
   paid."

This is deliberately the same pattern Nirium uses elsewhere for bounded
automation (a treasury-rebalance signer that has no destination
parameter to call at all): when a destination parameter *can't* be
removed from the call — paying different milestone receivers is the
entire point of an escrow — the authority boundary moves to *which
already-created object* the signer is allowed to touch, enforced by a
contract-address allowlist, rather than trying to make the destination
itself inexpressible.

**Tested, not just designed:** the allowlist's rejection path was run
live on testnet — an off-list contract was submitted and genuinely
blocked before the real, on-list contract was allowed through and
returned `SUCCESS` from both Horizon and Trustless Work's own escrow
record.

## Failure Modes

- **A tight settlement-signing window, found and resolved.** Signing and
  submitting a Soroban authorization inside Trustless Work's normal
  `/helper/send-transaction` flow left an uncomfortably narrow window
  (well under two minutes) between building the transaction and
  submitting it. Fix, confirmed working, not a proposed change to the
  protocol: submit the signed XDR directly to Horizon (skipping the
  extra network hop `/helper/send-transaction` adds), then call
  `PUT /indexer/update-from-txhash` with the resulting hash to register
  the settlement in Trustless Work's own system after the fact. This is
  an existing, documented mechanism (the Indexer), not a future
  feature request.
- **A real integration bug, self-found and self-fixed, worth naming as a
  failure mode for anyone else building against this API:**
  `getEscrowByContractId` requires the `contractIds` parameter as an
  array (`contractIds[]`), per Trustless Work's own docs — passing a
  bare string silently fails. Not a protocol gap, a caller-side mistake
  easy enough to repeat.
- **Branch ambiguity as a security failure mode, not just a docs
  annoyance.** Trustless Work's smart-contract repo carries
  Single-Release and Multi-Release as separate branches with separately
  named release functions and different fee-destination guarantees (see
  Escrow Value). A dependency check that doesn't pin the exact branch in
  use can produce a real, wrong security conclusion — this happened once
  here internally before being caught.
- **What happens if a milestone receiver disappears or a milestone is
  contested — deliberately not automated.** The narrow-scoped signer has
  no path for this at all; it can only route to the human-held Dispute
  Resolver role. This is a design choice, not a gap: an automated signer
  empowered to also resolve disputes would defeat the point of keeping
  its authority narrow.

## Open Questions

- **Should the two primitives compose into one caller-facing flow, or
  stay two separate integrations a requester chooses between?** Nirium
  has both working independently; neither has been wired so that an
  `x402` 402 response can itself carry escrow terms for a larger,
  protected principal, as RFC 0003 asks.
- **If composed, what triggers the handoff from direct payment to
  escrow?** A price threshold? An explicit flag in the payment
  requirement? Something the requester chooses per request?
- **Does the narrow-scoped-signer / allowlist pattern generalize as a
  reusable primitive across escrow use cases** (agent-hires-agent,
  enterprise procurement), or is it specific to the
  milestone-payout shape where destinations are fixed at creation time?
- **How should the branch-ambiguity failure mode above be prevented at
  the protocol/SDK level**, rather than relying on every integrator to
  independently discover and pin the correct contract variant?
