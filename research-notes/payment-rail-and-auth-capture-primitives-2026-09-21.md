# Payment Rail and Auth-Capture Primitives - 2026-09-21

Status: Research reference  
Scope: direct payment, metered payment, and authorization/capture primitives  
Normative status: non-normative; does not rewrite or approve any RFC

## Purpose

This note collects references for machine-native payment execution and payment-hold primitives that are adjacent to Trustless Work's agentic escrow research.

These systems may solve direct payment, session payment, usage metering, or delayed capture. They should not be assumed to solve outcome-conditioned agreement settlement without further analysis.

## Reference Map

| Source | What it contributes | Research caveat |
| --- | --- | --- |
| x402 v2 specification | HTTP-native payment negotiation with multiple schemes, including `exact`, `upto`, and `auth-capture` | Corrects the shorthand that x402 is only immediate payment |
| x402 `upto` | Authorizes a maximum and settles actual usage after consumption | Usage/metering escrow, not qualitative fulfillment escrow |
| x402 `auth-capture` | Authorization followed by later capture using Commerce Payments Protocol | Overlaps basic delayed capture; needs comparison with TW roles and dispute model |
| Commerce Payments Protocol | Onchain authorize, capture, charge, void, reclaim, and refund lifecycle | Strong payment-hold primitive; narrower than a multi-role agreement model unless extended |
| Shopify CPP explanation | Product framing for why delayed commerce needs programmable authorization/capture | Company-authored explanation; useful for product framing, not independent adoption proof |
| Machine Payments Protocol | Machine payment charges and sessions for high-frequency flows | Payment rail/session primitive, not agreement settlement |
| Solana payment channels | Escrowed spending ceiling with off-chain vouchers and settlement of actual usage | Supports metering/session settlement; not necessarily outcome evaluation |
| Stellar agentic payments docs | Documents x402 and MPP in the Stellar ecosystem | Relevant to composition with Stellar-based Trustless Work flows |
| AWS AgentCore Payments | Cloud-agent payment execution using x402/MPP-style primitives | Validates agent payment execution as infrastructure, but not outcome escrow by itself |

## Working Boundary

Payment rails answer:

```text
can the agent/client pay?
can payment be requested?
can a maximum be authorized?
can actual usage be settled?
can a merchant capture an authorized amount?
```

Trustless Work's strongest research question is different:

```text
should the counterparty receive committed funds yet?
what outcome was promised?
who verifies it?
what happens if fulfillment fails?
who can dispute, release, or recover?
```

## Open Analysis Questions

- Where does x402 `auth-capture` functionally overlap Trustless Work Single Release?
- Does CPP have enough role separation for platform, approver, release signer, and dispute resolver workflows?
- When should a direct payment, `upto`, MPP session, or auth-capture flow explicitly bypass Trustless Work?
- Which recovery semantics from payment rails should feed RFC 0004's agent-facing error model?

## References

- <https://github.com/x402-foundation/x402/blob/main/specs/x402-specification-v2.md>
- <https://github.com/x402-foundation/x402/blob/main/specs/schemes/upto/scheme_upto.md>
- <https://github.com/x402-foundation/x402/blob/main/specs/schemes/auth-capture/scheme_auth_capture_evm.md>
- <https://github.com/base/commerce-payments>
- <https://shopify.engineering/commerce-payments-protocol>
- <https://stripe.com/blog/machine-payments-protocol>
- <https://github.com/solana-foundation/payment-channels>
- <https://developers.stellar.org/docs/build/agentic-payments>
- <https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/payments-process-payment.html>
