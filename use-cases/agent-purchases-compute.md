# Agent Purchases Compute

## Summary

An agent buys compute, data, model inference, or API access from a machine-native service.

Some purchases may be immediate and low-risk enough for direct payment. Others may require escrow if delivery, quality, duration, or output must be verified.

## Actors

- Requesting agent
- Human or organization principal
- Compute/API provider
- Provider agent
- Verifier or policy system

## Flow

```text
Agent requests service
Provider returns payment requirement
Agent evaluates direct payment vs escrow
Small immediate fee may be paid directly
Larger conditional amount may enter escrow
Provider performs work
Evidence or output is evaluated
Escrow releases or disputes
```

## Why Direct Payment May Be Insufficient

Direct payment works for simple access. It is weaker when the buyer needs assurance that a longer-running job completed correctly or met required service levels.

## Escrow Value

- protects larger compute commitments;
- ties payment to completion;
- supports service-level evidence;
- enables refund or dispute if work fails;
- separates access fees from final settlement.

## Authority Model

Agents may be allowed to make small direct payments. Escrow funding and release should depend on amount, provider reputation, and verification confidence.

## Failure Modes

- job fails after payment;
- output quality is poor;
- provider overstates completion;
- verifier is unavailable;
- cost exceeds budget;
- agent loops on paid requests.

## Open Questions

- What compute outputs can be machine-verified?
- Should x402 handle the request payment and escrow handle the completion payment?
- How should rate limits and budgets bind to escrow policy?
