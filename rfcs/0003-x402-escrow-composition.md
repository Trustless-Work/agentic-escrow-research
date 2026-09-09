# RFC 0003: x402 and Escrow Composition

Status: Draft

## Summary

This RFC explores how direct machine-native payments and escrow-protected payments may compose.

The working assumption is that x402-style flows and escrow are complementary, not mutually exclusive.

## Direct Payment Is Suitable When

- payment is low-value;
- fulfillment is immediate;
- access can be granted automatically;
- refund or dispute risk is low;
- the buyer accepts finality;
- the service is digital and verifiable at request time.

Examples:

- pay-per-API call;
- small data purchase;
- content access;
- compute request with immediate response.

## Escrow Is Suitable When

- fulfillment happens later;
- delivery quality is uncertain;
- the amount is meaningful;
- the counterparty may fail to perform;
- evidence must be reviewed;
- human or policy approval matters;
- dispute or fallback paths are needed.

Examples:

- physical goods purchase;
- freelance task;
- enterprise procurement;
- AI agent hiring another service provider;
- grant or bounty payout;
- milestone-based work.

## Composition Pattern

```text
x402 payment request
  -> immediate access fee or payment requirement
  -> escrow terms for protected principal
  -> escrow creation / funding
  -> fulfillment and evidence
  -> approval / release
```

## Research Questions

- Can a payment requirement include escrow terms?
- Can a direct machine payment pay for quote generation while escrow protects fulfillment?
- Should x402 handle small access fees while escrow handles larger conditional principal?
- How should an agent decide between direct payment and escrow?
- What metadata should pass between the payment request and escrow creation?

## Non-Goals

- claiming escrow replaces x402;
- requiring every agentic payment to use escrow;
- designing a universal commerce protocol before use cases are validated.
