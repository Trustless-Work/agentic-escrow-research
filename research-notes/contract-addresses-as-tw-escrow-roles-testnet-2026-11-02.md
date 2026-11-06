# Contract Addresses as Trustless Work Escrow Roles - Stellar Testnet Evidence - 2026-11-02

Status: Research reference
Scope: Soroban contracts holding Trustless Work single-release escrow roles; receipt-gated release with deadline refund for agent payments
Informs: [RFC 0002](../rfcs/0002-agent-authority-levels.md) (authority levels, Level 3), [RFC 0001](../rfcs/0001-agentic-escrow-flow.md) (escrow flow), [RFC 0003](../rfcs/0003-x402-escrow-composition.md) (x402 composition)
Evidence class: testnet only; one implementation; unaudited prototype; not production evidence
Normative status: non-normative; does not propose a change to Trustless Work contracts

## Question

Can a Soroban **contract address** (not a classic `G...` account) hold the Approver, Release Signer, Platform and Dispute Resolver roles of a Trustless Work single-release escrow, and drive `approve_milestone`, `release_funds`, `dispute_escrow` and `resolve_dispute` itself?

If yes, the release rule of an escrow can be code with a narrow interface instead of a key held by an agent or an operator. That is relevant to RFC 0002 Level 3 ("agent can perform selected actions within explicit policy limits"): the agent never holds a role key on the escrow.

## Short answer

Yes, for the single-release contract version listed below, on testnet. All four role calls succeeded with a contract as the role holder, without changing the Trustless Work contract. Multi-release was not tested.

## Version context

| Item | Value |
| --- | --- |
| Network | Stellar testnet, 2026-11-02 |
| Trustless Work escrow wasm | `7c3f7b2af92ad86092708b23babf80f9e1308d7f3ce18b703b9499192ecc934b` (single-release; same hash as several TW escrows already on testnet, found via RPC `getEvents` on the `tw_init` topic; soroban-sdk 26, protocol 26) |
| Source read | role and dispute checks quoted here were read in `Trustless-Work/trustlesswork-smart-contract-stellar`, branch `single-release-develop` (2026-11-02); we matched its interface to the deployed wasm but did not rebuild the wasm hash from source |
| How it is used | deployed by hash from a contract (`deploy_contract`), then `initialize_escrow`; the TW wasm is not modified or redistributed |
| Token | Circle testnet USDC SAC `CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA` |
| TW fee address passed to `release_funds` / `resolve_dispute` (**observed, not official**) | `GA6KH5VWPCHBOEF63X57SPX6T4H366YFFKKGCVDBTXT2N7JVL6PJCK7G` (observed as the receiver of the 0.3% fee in existing TW testnet releases; pinned by us, not confirmed or provided by Trustless Work; the authoritative per-network value is to come from TW) |
| Test implementation | [AgentLeash Escrow](https://github.com/ttimesai-star/agentleash-escrow) commit `250a428`, MIT |

## Setup

Two small contracts:

- **LeashVault** holds the agent's funds. The owner sets an allow-list of sellers, a per-call cap, a window limit and an agent key expiry. The agent key can call only `pay(seller, amount, request_hash, timeout)`.
- **LeashResolver** holds the Dispute Resolver role. It obeys only its Vault (`vault.require_auth()`). It exists because the dispute resolver cannot open a dispute (`DisputeResolverCannotDisputeTheEscrow`) and only the dispute resolver can call `resolve_dispute`, so one address cannot both open and resolve a dispute.

Roles written into every escrow by `pay`:

| TW role | Holder |
| --- | --- |
| approver | LeashVault (contract) |
| release_signer | LeashVault (contract) |
| platform | LeashVault (contract), `platform_fee = 0` |
| dispute_resolver | LeashResolver (contract) |
| service_provider, receiver | seller (classic account) |

One milestone per escrow; one escrow per payment.

## How the calls are authorized

Soroban treats the direct invoker as authorized: when contract A calls contract B and B calls `A.require_auth()`, the check passes without any signature. All role checks inside the TW escrow are `require_auth()` on the role address passed in, so:

| Call | Invoker | Role checked | Extra auth needed |
| --- | --- | --- | --- |
| `initialize_escrow(props)` | Vault | none beyond the props | no |
| `fund_escrow(signer = Vault, ...)` | Vault | Vault | **yes**: the escrow calls `token.transfer(Vault -> escrow)`; Vault is not the direct invoker of that nested call, so the Vault pre-authorizes it with `env.authorize_as_current_contract([transfer(Vault, escrow, amount)])` |
| `approve_milestone(0, approver = Vault)` | Vault | Vault | no |
| `release_funds(release_signer = Vault, tw_fee)` | Vault | Vault | no (escrow transfers its own balance) |
| `dispute_escrow(signer = Vault)` | Vault | Vault (approver) | no |
| `LeashResolver.resolve(...)` | Vault | Vault, inside the resolver | no |
| `resolve_dispute(dispute_resolver = Resolver, tw_fee, {to: balance})` | Resolver | Resolver | no |

The only signature on any of these transactions is the outer one: the agent's on `pay`, or the transaction source's on `settle` / `refund`. `settle` and `refund` are permissionless entry points; the Vault's own checks (receipt signature, deadline, payment status) decide whether a TW call is made.

The Rust tests run against the real TW wasm with authorization enforced (`env.set_auths(&[])` after setup, no blanket `mock_all_auths` in the flows). A test also shows the seller cannot approve or release its own payment.

## Pattern: receipt-gated release with deadline refund

```text
agent  -> Vault.pay(seller, amount, request_hash, timeout)
          policy check -> deploy TW escrow by wasm hash -> initialize_escrow -> fund_escrow
seller -> serves the request, signs receipt = ed25519(seller_key,
          sha256("ALSH-RCPT-v1" || xdr(vault) || xdr(escrow) || request_hash || result_hash))
anyone -> Vault.settle(escrow, result_hash, sig)   if now <= deadline
          -> approve_milestone + release_funds -> seller
anyone -> Vault.refund(escrow)                     if now > deadline
          -> dispute_escrow -> Resolver.resolve -> resolve_dispute -> Vault
```

The seller's receipt key is recorded at `pay` time, so later changes to the allow-list do not change which key can settle an open payment. If the seller opens a dispute itself, a valid receipt still pays it, through the resolver.

What the receipt proves: the seller committed to a specific `result_hash` for a specific escrow and request. It does **not** prove the result is correct or useful. This is an outcome-commitment gate, not a quality verifier; a buyer-side or third-party check would be a different release condition.

## Results (testnet, 2026-11-02)

LeashVault [`CANKRX7T4U7PBN7SBXSJHX5Y5FKQCCWRXSHPJL6ZCJS4TILNSC6VC6RH`](https://stellar.expert/explorer/testnet/contract/CANKRX7T4U7PBN7SBXSJHX5Y5FKQCCWRXSHPJL6ZCJS4TILNSC6VC6RH), LeashResolver [`CAH2BXINTHA5OT2Z5LGSD4QUGFUEYHR7SQEH5YA5LUXBFN7RVQHJOQ3G`](https://stellar.expert/explorer/testnet/contract/CAH2BXINTHA5OT2Z5LGSD4QUGFUEYHR7SQEH5YA5LUXBFN7RVQHJOQ3G).

| Flow | TW escrow | Transactions | Outcome |
| --- | --- | --- | --- |
| release, seller answers HTTP 402 and settles itself | [`CBCA2TMI...YTDE`](https://stellar.expert/explorer/testnet/contract/CBCA2TMI5KPBQ5J4IUTUFRY7Y2R53Y7MQ2SA7EWAQLEM2BFVWQ37YTDE) | pay [`d4120cd3...`](https://stellar.expert/explorer/testnet/tx/d4120cd3fa49c028da41cee229302624f817093bfa72090a38023570e348b3fd), settle [`687007ad...`](https://stellar.expert/explorer/testnet/tx/687007ad98fbf8d28780dcb5cdf6dae7c573ed8c20da92462f02e4b8212a3bd4) | seller +1.994 of 2 USDC |
| release | [`CCTSFQVS...3NI2`](https://stellar.expert/explorer/testnet/contract/CCTSFQVSL6ML4TBHK46YALGUPG6ECBIXNO637CSPRF72ARODP2HN3NI2) | pay [`21854073...`](https://stellar.expert/explorer/testnet/tx/218540736f0238b88f7889f9690b1d3d0428ae338c6b4b9fa1d42dff5a35add0), settle [`5ac5d0a3...`](https://stellar.expert/explorer/testnet/tx/5ac5d0a325b02916f24a0b2f1ffcc0f83c9a274fcf013b321f3dac67ca84cae3) | seller +1.994 of 2 USDC |
| refund after 60 s deadline (dispute -> resolve) | [`CDRZHC5Y...67VQ`](https://stellar.expert/explorer/testnet/contract/CDRZHC5YNH3MPTRE4SQGXBOSFMBLIYEP3LBQRIAYTXKCL2LA5KJP67VQ) | pay [`1a37fe0a...`](https://stellar.expert/explorer/testnet/tx/1a37fe0a518ce651163fdfb2c300d4f78d68b6ff49648f9fe6b65eb339115d31), refund [`5183d063...`](https://stellar.expert/explorer/testnet/tx/5183d0634362417cdd5922de87729de35cf3d9da7af28a5354ab1ff186f33a04) | Vault +0.997 of 1 USDC; valid receipt after the deadline -> `Error(Contract, #11)`; after the refund -> `Error(Contract, #10)` |
| forged receipts | [`CCAMTDL5...YYSG`](https://stellar.expert/explorer/testnet/contract/CCAMTDL57E4EN24KWIIXOJVEXTONVS42L35U7NKOTC7W7U5RNLX3YYSG) | pay [`f25c336d...`](https://stellar.expert/explorer/testnet/tx/f25c336d6dbf93fc19b837dc896f1cd82a9e422255d77801454ba4f1f44c3d11), genuine settle [`af7fdcf5...`](https://stellar.expert/explorer/testnet/tx/af7fdcf548c52196d4d9689967e22d7e5d6b3df6887abf96bfe9876c45dcaef0) | wrong key, other `result_hash`, receipt of another escrow -> `Error(Crypto, InvalidInput)`, nothing released; genuine receipt settles |
| policy refusals | - | revoke [`a49eb9df...`](https://stellar.expert/explorer/testnet/tx/a49eb9dfea0bd168cc00f3fff7dc3c7f82716f8fbe8265367db55ca3d95c34ad), restore [`53ac622b...`](https://stellar.expert/explorer/testnet/tx/53ac622b0722d8ae6ebd4866f6f4ff8f2624eb6e3a3f9e9d26e0908782576aa3) | seller not allow-listed `#3`, over per-call cap `#5`, over window `#6`, revoked agent `#2`: the SDK refuses before signing; forced to chain, the contract returns the same codes; no escrow is created |

17 transactions in this deployment, all `successful: true` on Horizon testnet. All payments are between our own test keys: this is internal testing, not usage. Full record: [`deployments/testnet.json`](https://github.com/ttimesai-star/agentleash-escrow/blob/250a428ba7e676e8c76179adae377a3501958a4d/deployments/testnet.json).

## Observations relevant to Trustless Work

1. **Contract-held roles work without contract changes.** Nothing in the single-release escrow distinguishes contract addresses from accounts for role checks. Approver, Release Signer, Platform and Dispute Resolver can all be contracts.
2. **A contract that wants its own return path needs a second address.** The dispute resolver cannot open a dispute and only the dispute resolver can resolve, so a single contract cannot run dispute -> resolve alone. Any other role, including the service provider, can open a dispute; the Vault handles that case (a valid receipt still pays the seller through the resolver; after the deadline the payer gets the refund).
3. **No timeout in the escrow itself.** The single-release interface used here has no expiry entry point. A deadline refund has to be built outside the escrow; here it is `dispute_escrow` + `resolve_dispute` with the whole balance to the payer. On-chain, a refund therefore looks like a resolved dispute, not a cancellation. Indexers and dashboards that count disputes would count every timeout refund as one.
4. **The 0.3% protocol fee applies on both paths.** `resolve_dispute` back to the payer also pays the fee: a refund returns 99.7%. For small or frequent agent payments where refunds are expected, this is a cost of every non-delivery, not only of completed sales.
5. **The fee address is a caller argument.** `release_funds` and `resolve_dispute` take the TW fee address from the caller. Our contract pins the address **observed** in TW testnet releases; it is not an official or confirmed value. A contract integrator needs an authoritative per-network value from Trustless Work.
6. **Indexing is unverified.** The escrows are deployed from the same wasm hash as TW's own, but we have no TW API key and did not check whether the TW indexer / `get-escrows-by-role` picks up escrows deployed by a third-party contract. They are visible on stellar.expert.

## Limits of this evidence

- Testnet only; one implementation; unaudited. One external AI reviewer (Google Jules, an AI code-review agent; one closed session, no public link) reviewed the contracts and scripts: 1 medium (an unchecked window addition trapped instead of returning the over-limit error) and 4 low (resolver TTL at deploy, withdraw of a non-positive amount, view simulations using a secret key, `set_limits` with a shorter period resetting the window). The medium and three lows were fixed, each with a failing test first, before this deployment; the `set_limits` item was left unchanged (owner-only, documented). A second model (Gemini) raised two "high" findings, both disproved by auth-enforced tests and testnet transactions. Jules output is a review aid, not an audit.
- Single-release only. **Multi-release (per-milestone dispute and resolution) was not tested.**
- One escrow contract is deployed per payment. Cost and ledger footprint at volume were not measured.
- The demo seller ran on localhost in the same process as the agent; there are no external sellers or buyers.
- The receipt binds a result hash, not result quality (see above).

## Open questions

1. Does Trustless Work intend contract addresses as role holders to be a supported configuration, or an incidental property of the current single-release contract?
2. Is per-milestone dispute / resolution in multi-release equally usable by a contract resolver, and can a resolver split a single milestone between payer and payee?
3. Would a native deadline / expiry path (refund without a dispute record and, possibly, without the fee) fit the "keep the primitive narrow" principle, or should timeout refunds stay outside the escrow?
4. What is the authoritative fee address per network for contracts that call `release_funds` / `resolve_dispute` directly?
5. Are escrows deployed by third-party contracts from the approved wasm hash meant to be indexed and returned by role queries?

## Mapping to RFC 0002

This is one way to reach Level 3 without giving the agent a role key on the escrow: the agent holds a key that can only call `pay` within owner-set limits; the economic signer for release and refund is a contract whose rule is public and fixed at deploy; the owner keeps `revoke_agent`, `set_limits` and `withdraw`. The authority boundary is enforced twice: pre-sign in the agent SDK and again on-chain, with the same error codes.
