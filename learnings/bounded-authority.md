# Bounded authority: narrow Level 3 execution

**ID:** `authority/bounded-execution`  
**Status:** research-supported  
**Confidence:** limited  
**Related RFCs:** [0002](../rfcs/0002-agent-authority-levels.md), also informs [0001](../rfcs/0001-agentic-escrow-flow.md)  
**Example:** [../examples/permission-liveness-last-action.json](../examples/permission-liveness-last-action.json)

## Question

How can an agent automate escrow or payment actions without receiving open-ended economic authority?

## Lesson

Safe autonomy comes from **bounded execution inside a human-defined commitment**, not from giving the agent a general-purpose wallet.

Durable patterns in this repository:

1. **Human defines the economic boundary; automation operates inside it.** A human fixes receivers, amounts, milestones, and roles. A narrow signer may approve or release only on allowlisted, already-created escrows.
2. **Delegate execution before economic destination-setting.** Choosing who gets paid is a stronger authority than releasing an already-fixed payout.
3. **Make unauthorized actions inexpressible.** Prefer interfaces and contracts that omit arbitrary beneficiary parameters over policy that merely forbids them.
4. **Separate exfiltration risk from economic-decision risk.** Blocking arbitrary withdrawals does not by itself prevent loss through poor timing, allocation, or churn.
5. **Agent-side policy is not on-chain enforcement.** A pre-sign policy hook can fail closed before signing, bind approval to exact authorization bytes, and expire or stale-out decisions. Same-process signer access and concurrent capacity races still need account- or contract-level controls.
6. **Contract-held escrow roles are a promising Level 3 shape on testnet.** In one single-release experiment, Soroban contracts held Approver, Release Signer, Platform, and Dispute Resolver on unmodified Trustless Work wasm, so the agent never held a role key. This is **testnet evidence for one implementation**, not a supported-product declaration.

## Evidence

| Claim | Class | Source |
| --- | --- | --- |
| Narrow automated approve/release on allowlisted multi-release escrows | production-tested use case | [Nirium bounded milestone payouts](../use-cases/nirium-bounded-milestone-payouts.md) |
| Authority by missing beneficiary parameter | production-tested use case | [Nirium treasury missing parameter](../use-cases/nirium-treasury-missing-parameter.md) |
| Evidence checks before approve/release; bad evidence refused | testnet; decision-step code private | [Evidence-gated release pilot](../research-notes/evidence-gated-release-testnet-pilot-2026-10-14.md) |
| Pre-sign policy outcomes; fail closed; approval bound to exact bytes | implementation + tests; no production use claimed | [Pre-sign policy hook](../research-notes/pre-sign-policy-hook-2026-10-21.md) |
| Contract addresses as TW escrow roles; receipt-gated release; deadline refund via dispute | testnet only; single-release; unaudited prototype | [Contract-held roles](../research-notes/contract-addresses-as-tw-escrow-roles-testnet-2026-11-02.md) |
| Fail-open on internal error is an authority failure | mixed: x402 gate affected a mainnet route; delivery bug was testnet; some fix commits private | [Fail-closed gates](../research-notes/fail-closed-payment-and-delivery-gates-2026-10-05.md) |

## Limitations

- RFC 0002 originally recommended targeting Level 1 or 2 for first public experiments. Later evidence supports **narrow Level 3 execution** as research-supported, not as a general recommendation to automate approval and release together.
- Contract-held roles: multi-release parity untested; indexer discovery of third-party-deployed escrows unverified; fee address used in the note is observed, not official.
- Evidence-gated pilot: checks integrity, addressing, and provider signature; does not prove the work is true or useful. Final-rule allow path not shown on-chain.
- Pre-sign hook does not stop same-process raw signer use.

## Remaining design decision

How should delegated authority scope be made explicit, enforceable, inspectable, and revocable—especially below the SDK layer—and does Trustless Work treat contract-held roles, including multi-release behavior, as a supported configuration?
