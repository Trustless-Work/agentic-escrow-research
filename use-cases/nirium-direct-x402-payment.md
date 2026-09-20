# Direct Payment Without Escrow: A Verified Live x402 Settlement

## Summary

The two use cases already in this repo (`nirium-bounded-milestone-payouts.md`,
`nirium-treasury-missing-parameter.md`) both describe *bounded automated
authority* — a signer that is deliberately narrow. This one describes the
other end of RFC 0003's spectrum: **no escrow, no bounded signer, no
authority question at all**, because the payment is small, digital, and
settles atomically before anything of value is handed over.

The primitive is `x402Serve()`, from the open-source
[`nirium`](https://github.com/nirium-protocol/nirium-sdk) SDK, published on
Stellar's official developer skills catalog as
[`nirium-agentic-payments`](https://github.com/nirium-protocol/nirium-sdk/blob/main/skills/nirium-agentic-payments/SKILL.md)
(reviewed and merged by SDF DevRel:
[stellar/stellar-dev-skill#96](https://github.com/stellar/stellar-dev-skill/pull/96)).
Nirium's own mainnet endpoint runs the identical function in production. One
of its settlements, dated 9 July 2026, was independently re-verified for
this document directly against Horizon rather than taken from the skill's
own claim — see Flow below for the exact record.

This document is deliberately narrower in scope than the first two: it is
not proposing that direct payment is *sufficient* everywhere, it is
documenting the specific shape RFC 0003 already names as the direct-payment
case, with real evidence, and then naming the failure modes that showed up
in production anyway — which turned out to be about a dependency (the
facilitator), not about the counterparty, which is the risk escrow is built
to cover.

## Actors

| Actor | Role |
| --- | --- |
| **Caller** | An AI agent or any HTTP client requesting a paid route. Holds the USDC it pays with and signs its own transfer authorization. |
| **Operator** | Runs `x402Serve()` on their own server, sets the price per route and the `payTo` address in their own deployed code. In the verified transaction below, this is Nirium's own production endpoint. |
| **x402 Facilitator** | A third party (OpenZeppelin Channels, in this deployment) that verifies the caller's payment and relays it to the network. Per Nirium's own published skill and devlog, this is an external dependency Nirium does not operate. |
| **Stellar Network / USDC SAC contract** | Executes the transfer atomically once a valid authorization is submitted. |

## Flow

```text
Caller requests a paid route with no payment attached
  -> operator's x402Serve() returns HTTP 402 carrying the route's
     terms (price, asset, payTo) in the payment-required challenge
  -> caller signs a Soroban authorization for the exact amount and
     retries the request with payment attached
  -> facilitator verifies the payment and submits it to Stellar
  -> USDC SAC transfer() executes; operator's payTo balance
     increases atomically with settlement
  -> operator's route handler returns the paid data
```

**The verified transaction.** Nirium's skill claims a mainnet settlement
from 9 July 2026 at hash `3134a51c…7558bc`. Rather than cite that as given,
it was looked up directly against Horizon for this document:

- Hash: [`3134a51c66091fd7fbd85b38a4a6ec6cd432bb92c2450eac84ea7855cb7558bc`](https://stellar.expert/explorer/public/tx/3134a51c66091fd7fbd85b38a4a6ec6cd432bb92c2450eac84ea7855cb7558bc) — `successful: true`, ledger `63403784`, `created_at: 2026-07-09T20:24:34Z`.
- One operation: `invoke_host_function` calling `transfer` on the USDC SAC
  (issuer `GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN`, Circle's
  canonical Stellar mainnet USDC issuer), moving **0.0200000 USDC** — which
  matches the exact `'GET /signals': '$0.02'` example route the skill
  itself documents, not a round or coincidental number.
- It is a fee-bump transaction: `fee_account` is
  `GA5SXMFJTUPTZRIEKM6XZLCYOZRMUEE6KGAHL3GXDBG64DYOUIWYIF3M`, distinct from
  the address whose USDC balance actually moved
  (`GAQPWCSBHQNBHR6XXAO2F3MNZIKWPDHCKNU5IEQ6PX6YEIUNNGWE252E`). That
  `GA5SXMFJ…` prefix matches the facilitator sponsor account Nirium's own
  devlog names repeatedly in its facilitator-outage entries — an
  independent cross-check between two separately-fetched sources (a live
  Horizon record and a static devlog page), not something asserted from
  either one alone.

This confirms the settlement is real and matches the skill's claim. It does
not, by itself, confirm every operational detail in the rest of this
document — those are sourced from Nirium's own public devlog and marked as
such below, not re-verified independently the way the transaction was.

## Why Direct Payment May Be Insufficient

By RFC 0003's own criteria, this case is squarely in the direct-payment
column: low value, immediate digital fulfillment, no delivery-quality
uncertainty, finality accepted by both sides. The RFC is right that escrow
adds nothing here for *counterparty* risk — the caller gets exactly the
paid-for response or nothing, atomically, in one settlement.

What the RFC's decision tree doesn't name as a separate axis is
**dependency risk**, and Nirium's own devlog documents two real incidents
on this exact production path that fall on that axis instead:

- **5 August 2026 — a rejected payment settled anyway.** The endpoint
  returned HTTP 402 (payment rejected) to a caller, and roughly 30 seconds
  later that same payment settled on-chain regardless, from a different
  relayer-pool account than the one that settled the caller's separate
  retry. The caller was charged and told they had not paid. Diagnosed with
  a third-party integrator (AgentLedger) who reproduced the same failure
  signature against two unrelated x402 sellers on Stellar pubnet, pointing
  at the facilitator layer, not any one seller's integration. Filed as
  [x402-foundation/x402#3148](https://github.com/x402-foundation/x402/issues/3148).
- **10 August – 11 September 2026 — a ~32-day facilitator outage.** The
  hosted facilitator instance was pointed at a stale RPC URL, so `verify()`
  threw before evaluating any payload — every request on this route failed
  for the entire window, for reasons entirely outside the operator's code.
  Root cause and fix landed with the facilitator's own maintainer at
  [OpenZeppelin/relayer-plugin-x402-facilitator#47](https://github.com/OpenZeppelin/relayer-plugin-x402-facilitator/issues/47).

Both are self-reported in Nirium's own devlog — read directly for this
document, not independently reproduced the way the transaction hash was.
Both are still genuine evidence that "direct payment is the correct choice
here" is not the same claim as "this path has no failure modes."

## Escrow Value

Escrow would not have helped with either incident above, and that is worth
stating precisely rather than skipped past. Escrow protects against a
*counterparty* failing to perform after being paid — it has nothing to say
about a payment *infrastructure dependency* being down or racy, because
that dependency sits underneath both the direct-payment path and any
escrow-based one equally. Holding $0.02 in an escrow contract while a
facilitator's RPC endpoint is misconfigured does not get the caller their
data any sooner, and does not give them a dispute path a well-designed
reconciliation and refund process couldn't already provide.

What this use case actually argues for — which is a narrower and more
honest claim than "escrow isn't needed" — is that **RFC 0003's
direct-vs-escrow axis and a dependency-reliability axis are orthogonal**.
A flow can be correctly placed in the "direct payment is sufficient" column
by every criterion the RFC lists, and still need its own answer to "what
happens when the facilitator, not the counterparty, is the thing that
fails" — an answer escrow was never going to provide.

## Authority Model

There is no bounded-signer question here, unlike the two existing use
cases in this repo — and that absence is the point being documented, not
an oversight:

- The caller signs their own Soroban authorization for the exact amount
  it's paying. No other party can construct a valid transfer on the
  caller's behalf.
- `payTo` is a value the operator hardcodes in their own deployed
  `x402Serve()` call — not something the caller or the facilitator can
  redirect. In the verified transaction, the destination
  (`GCLBBPON256CV7ATEHM5B54BOKNC7GX53MBINJ42MHVXGDMMZ3ZWKBHP`) is exactly
  the address named in Nirium's own published skill as their production
  `payTo`; this document did not find an independent public directory tag
  confirming that address's label, so it is sourced to the skill, not to
  a third-party registry.
- The facilitator's role, per the fee-bump structure of the verified
  transaction, is limited to relaying and sponsoring the network fee — it
  is a distinct account from the one whose USDC balance moved, consistent
  with "verifies and submits" rather than "custodies or redirects."
- No party holds a balance in trust at any point. There is nothing to
  rebalance, release, or dispute, because the transfer and the service
  delivery are the same atomic event.

## Failure Modes

- **Charged with no recourse when the facilitator itself misbehaves.** The
  5 August race condition (above) is the sharpest example: a caller can be
  billed and told simultaneously that they were not billed, with no
  protocol-level dispute path, because there is no escrow layer to fall
  back into. Nirium's own fix was off-protocol — a reconciliation endpoint
  (`GET /api/reporting/reconcile`) comparing on-chain settlements to logged
  receipts after the fact, not a real-time recourse mechanism.
- **The reconciliation fix has a known, acknowledged gap.** Per the same
  devlog entry, receipts logged before that endpoint existed don't carry
  the settlement hash, so historical charges can't all be matched
  transaction-to-receipt one-to-one — only in aggregate, until enough time
  passes for hash-carrying receipts to dominate the history.
  Self-acknowledged by Nirium, not found independently.
- **A correctly-designed direct-payment route is still fully dependent on
  a facilitator it doesn't operate.** The 32-day outage (above) didn't
  expose a bug in `x402Serve()` or in the caller's client — the entire
  route was simply unusable for over a month because of an upstream
  misconfiguration neither Nirium nor the caller controlled, with no
  fallback path available inside the direct-payment model itself.

## Open Questions

- **Does RFC 0003's decision tree need a second axis?** Value, timing, and
  counterparty risk decide direct-vs-escrow; dependency reliability
  (is the facilitator itself healthy) is a separate question this use case
  suggests the RFC doesn't currently ask, and escrow doesn't answer either.
- **Should a caller — especially an unattended agent — be able to check
  facilitator health before retrying, the way escrow gives it a queryable
  on-chain milestone state to check?** Today that information exists only
  as a GitHub issue thread, not as anything machine-readable at request
  time.
- **Is a standard reconciliation/receipt schema across x402 sellers a
  primitive worth this repo's attention**, given that Nirium built its own
  ad hoc version after the fact and the fix has an acknowledged historical
  gap? Or is that out of scope for a repo about escrow composition
  specifically?
- **Does "who pays the network fee" matter to this repo's authority
  model the way "who can redirect funds" already does?** The verified
  transaction shows the facilitator sponsoring the fee via a fee-bump,
  structurally separate from the principal transfer — worth naming
  explicitly even though, unlike the other two use cases here, no party
  in this flow custodies anything at all.
