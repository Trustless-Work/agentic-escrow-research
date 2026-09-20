# Authority By Missing Parameter: An Agent That Cannot Externalize Funds

## Summary

A companion use case to the bounded-authority milestone-payout pattern
already in this repo, describing a structurally stronger authority
guarantee: an automated signer whose only callable action **has no
destination argument at all**. This isn't "restricted to an allowlist" —
it's "the function the signer can call is physically incapable of
sending funds anywhere outside one client-owned vault," verified by
reading the deployed contract's source directly, not assumed from
documentation.

Built and run for real: Nirium's Treasury Node holds a `RebalanceManager`
role over a client-owned DeFindex vault (a Stellar/Soroban yield vault,
audited by OtterSec) and has moved real funds on mainnet using exactly
this authority shape.

## Actors

| Actor | Role |
| --- | --- |
| **Vault Owner** | Deploys, funds, and can withdraw from their own vault with their own signature. Full custody, never delegated. |
| **RebalanceManager (automated signer)** | A separate Nirium-held key, scoped by a Soroban policy account to a single `CallContract` rule on the one vault it manages — no `Default` rule, no broader capability. |
| **DeFindex Vault Contract** | The audited Soroban contract holding funds and exposing the `rebalance()` entrypoint |
| **Underlying Strategy** | The yield strategy the vault rebalances into/out of (in this deployment, an Etherfuse CETES strategy) |

## Flow

```text
Vault Owner deploys + funds their own vault (their own signature)
  -> RebalanceManager key is granted a scoped policy: CallContract
     on this one vault only, no other permission
  -> RebalanceManager calls rebalance() with an Invest/Unwind/
     SwapExactIn/SwapExactOut instruction
  -> funds move between "idle" and "invested" state INSIDE the vault
  -> Vault Owner can withdraw at any time with their own signature —
     the automated signer has no withdrawal capability at all
```

## Why Direct Payment May Be Insufficient

This isn't a payment-for-goods case, so the RFC 0003 direct-vs-escrow
framing applies differently here: there's no counterparty being paid at
all. The problem being solved is *recurring autonomous action over a
single pool of already-owned funds* — closer to "how much authority can
you safely delegate to automation that acts repeatedly, unsupervised,
over time" than to a one-shot payment decision. Named here because it's
a genuine third authority pattern alongside "one-shot direct payment"
and "human-created, bounded-release escrow": *recurring internal
rebalancing with no external payee at all.*

## Escrow Value

There's no escrow in this flow — worth stating precisely, since that's
the point. The safety property comes from a different mechanism:
**the contract's own function signature**, verified by reading
`rebalance()`'s implementation directly in the vault contract's source.
All four of its branches (`Invest`, `Unwind`, `SwapExactIn`,
`SwapExactOut`) encode their destination as the vault contract itself —
none accept an external address argument. A fully compromised
RebalanceManager key can move funds between idle and invested state
inside the one vault it's scoped to, and cannot direct them anywhere
else, because there is no parameter through which "anywhere else" could
be expressed.

## Authority Model

Two independent, stacked guarantees — either alone would already be
meaningful, together they don't share a single point of failure:

1. **Scope, at the key-management layer:** the RebalanceManager key
   is restricted by a Soroban policy account to exactly one
   `CallContract` rule, targeting the one vault it manages. No `Default`
   rule exists for this key.
2. **Capability, at the contract layer:** even without the policy
   restriction above, the called function itself has no destination
   parameter to misuse. This is the stronger of the two guarantees,
   because it doesn't depend on the key-management layer being
   correctly configured — it holds even under an assumption of "the
   scope check failed."

This is a meaningfully different shape from the milestone-payout use
case already in this repo, where paying different receivers is the
entire point of the flow and authority has to be bounded by *which
already-created object* a signer may touch. Here, the destination
parameter simply doesn't exist for any value the signer could supply —
a stronger, less caller-dependent guarantee, but only available when
the automated action genuinely never needs to name an external payee.

## Failure Modes

- **Vendor corroboration is not the same as on-chain verification, and
  the two should not be conflated when citing evidence.** The vault's
  security posture (same audited WASM release as the underlying
  strategy) was confirmed directly by the strategy's own maintainer.
  Separately, specific transaction links shared afterward were **not**
  corroborated by that maintainer — no reply was ever received. Both
  the underlying transactions and the vault contract remain
  independently verifiable on a block explorer regardless of vendor
  reply status, but a contributor citing this kind of evidence should
  keep "independently verifiable on-chain" and "corroborated by the
  counterparty" as two separate claims, not one.
- **Two real bugs found while integrating, reported to and fixed by the
  vault's maintainer** — worth naming as a failure mode anyone
  integrating with a similar vault/strategy pattern should watch for:
  a rounding mismatch between two different balance-reporting functions
  that could produce an `UnwindMoreThanAvailable` error on an
  otherwise-valid, round-number request, and a single error code shared
  by two different failure conditions (an empty vault vs. an empty
  strategy), making the two indistinguishable from the error alone.
- **No monetization is wired to this flow today** — relevant to anyone
  evaluating this as an economic model, not just a security model. The
  value demonstrated here is the authority pattern itself, not a
  revenue mechanism.

## Open Questions

- **Does "authority by missing parameter" generalize beyond
  single-vault rebalancing?** It works cleanly when the automated
  action never needs an external destination. Most agentic-commerce
  flows this RFC set is exploring (buying, hiring, paying bounties) do
  need one — does that mean this pattern is a narrow special case, or
  is there a broader class of agent actions that could be redesigned to
  not need a destination parameter at all?
- **Should a research track exist specifically for "recurring
  autonomous action over owned funds," separate from the
  buyer/seller-shaped use cases already in this repo?** This flow has
  no counterparty being paid — it may not fit cleanly into the
  agent-buys-X / agent-hires-X framing the existing use cases share.
- **How should a contributor's evidence distinguish "independently
  verifiable" from "vendor-corroborated" as a standard practice**, given
  how easy the two are to conflate in a research repository that
  explicitly asks contributors to separate facts from assumptions?
