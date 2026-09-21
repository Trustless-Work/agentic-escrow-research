# Conditional Settlement Taxonomy - 2026-09-21

Status: Research reference  
Scope: taxonomy scaffold  
Normative status: non-normative; does not rewrite or approve any RFC

## Purpose

This note preserves the working taxonomy from the internal context research so it can be reviewed inside the public research repository before any RFC-level changes are proposed.

The working distinction is that "escrow" in agentic commerce currently covers at least three different problems:

1. usage or metering escrow;
2. authorization/capture escrow;
3. outcome-conditioned agreement settlement.

## Taxonomy

| Category | Protected question | Representative systems | Relationship to Trustless Work |
| --- | --- | --- | --- |
| Usage escrow | How much was actually consumed? | x402 `upto`, MPP sessions, payment channels | Adjacent; useful for metered services |
| Authorization/capture escrow | How much may the merchant capture after authorization? | x402 `auth-capture`, Commerce Payments Protocol, ASP | Overlaps basic delayed capture and Single Release-style payment holds |
| Outcome/agreement escrow | Was agreed work or an outcome verified, and what recourse applies? | ERC-8183, VCAP, Sinetti, ENACT, Trustless Work | Core strategic research category |

## Usage Escrow Pattern

```text
deposit ceiling
-> consume service
-> settle actual usage
-> return or preserve remainder
```

This is primarily a metering problem. It asks "how much was used?" rather than "was the promised outcome fulfilled?"

## Authorization/Capture Pattern

```text
authorize amount
-> merchant fulfills
-> capture final amount
-> void / reclaim / refund remainder
```

This maps closely to delayed commerce and card-style authorization/capture. It protects against over-capture and enables later settlement, but it may not model richer agreement semantics by itself.

## Outcome/Agreement Pattern

```text
agreement
-> fund
-> perform work
-> submit evidence
-> evaluate / approve or dispute
-> release / refund / resolve
```

This is the category closest to Trustless Work's current role model and Multi Release behavior.

## Working Differentiation Hypothesis

Generic payment holds are becoming easier to express in payment protocols and chain-native tooling. Trustless Work's possible wedge is not "holding funds until later" by itself.

The stronger hypothesis to test is:

> Trustless Work coordinates programmable agreements, not just protected payments.

## Reference Inputs

| Source | Reference |
| --- | --- |
| x402 v2 specification | <https://github.com/x402-foundation/x402/blob/main/specs/x402-specification-v2.md> |
| x402 `upto` scheme | <https://github.com/x402-foundation/x402/blob/main/specs/schemes/upto/scheme_upto.md> |
| x402 `auth-capture` scheme | <https://github.com/x402-foundation/x402/blob/main/specs/schemes/auth-capture/scheme_auth_capture_evm.md> |
| Commerce Payments Protocol | <https://github.com/base/commerce-payments> |
| Machine Payments Protocol | <https://stripe.com/blog/machine-payments-protocol> |
| Solana payment channels | <https://github.com/solana-foundation/payment-channels> |
| ERC-8183 | <https://eips.ethereum.org/EIPS/eip-8183> |
| VCAP draft | <https://datatracker.ietf.org/doc/html/draft-stone-vcap-02> |

## Analysis To Do

- Decide whether the taxonomy should become a dedicated RFC amendment or remain research vocabulary.
- Compare the taxonomy against RFC 0003's direct-payment-versus-escrow decision tree.
- Identify which category each existing use case primarily belongs to.
