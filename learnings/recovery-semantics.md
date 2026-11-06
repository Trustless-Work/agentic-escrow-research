# Recovery semantics: settlement, delivery, and signer facts

**ID:** `recovery/settled-not-delivered`  
**Status:** research-supported  
**Confidence:** limited  
**Related RFCs:** [0004](../rfcs/0004-agent-facing-tool-schema.md); [0005](../rfcs/0005-agent-facing-tool-schema-production-evidence.md) is supporting evidence, not a competing design  
**Examples:** [../examples/payment-settled-not-delivered.json](../examples/payment-settled-not-delivered.json), [../examples/permission-liveness-last-action.json](../examples/permission-liveness-last-action.json)

## Question

After a payment or escrow action, what must an unattended agent be able to distinguish before it retries, waits, escalates, or assumes success?

## Lesson

Agents need a **minimal result and error envelope**, not only success or failure.

Distinctions supported by current evidence:

1. **Chain state versus read-model state.** A transaction can be submitted successfully while an indexer or application read still lags.
2. **Settled versus delivered.** Money can become final while the paid response or fulfillment is still missing. Treat “settled, not delivered” as its own recovery case.
3. **Who submits changes the failure mode.** If the client broadcasts first (push), verification failure can leave the payer charged with nothing delivered. If the server verifies before submit (pull / typical x402), failed verification leaves the payer uncharged.
4. **Operation existence versus caller capability.** Defining `release_payment` in a schema is not the same as exposing it to the current caller.
5. **Permission versus liveness versus last confirmed action.** “Enabled” / allowlisted does not mean the signer is running, and neither means it recently moved funds on-chain.
6. **Fail closed on internal error.** Payment verification and delivery authorization are separate gates. If a gate cannot evaluate, deny and return a machine-readable reason. Development fallbacks that pass through on missing config become authority bugs once real value is involved.

Field names such as `settlementState`, `retrySafe`, `dependencyState`, `readModelState`, and `nextValidActions` are **research vocabulary**, not approved API fields. See also the recovery scaffold in [agent-facing interface recovery semantics](../research-notes/agent-facing-interface-recovery-semantics-2026-09-21.md).

## Evidence

| Claim | Class | Source |
| --- | --- | --- |
| Candidate tool state and recovery envelope | RFC draft | [RFC 0004](../rfcs/0004-agent-facing-tool-schema.md) |
| Chain vs indexer lag; signer mode; caller capability; malformed input | research note supporting RFC 0004 | [RFC 0005 production evidence](../rfcs/0005-agent-facing-tool-schema-production-evidence.md) |
| Facilitator / infrastructure failures need machine-readable recovery even when escrow is not used | mainnet use case | [Nirium direct x402 payment](../use-cases/nirium-direct-x402-payment.md) |
| Settled ≠ delivered; submission mode matters | testnet; historical push failure unverifiable | [x402 vs MPP Charge](../research-notes/x402-vs-mpp-charge-testnet-2026-10-21.md) |
| Permission ≠ liveness ≠ last confirmed action | mainnet observation; reporting fix private | [Enabled is not activity](../research-notes/enabled-is-not-activity-2026-10-21.md) |
| Fail-open payment and delivery gates | mixed mainnet route + testnet delivery bug; some commits private | [Fail-closed gates](../research-notes/fail-closed-payment-and-delivery-gates-2026-10-05.md) |
| Pre-sign outcomes as retryable / terminal / dependency vocabulary | implementation + tests | [Pre-sign policy hook](../research-notes/pre-sign-policy-hook-2026-10-21.md) |

## Limitations

- No shared standard recovery envelope is adopted across direct rails and Trustless Work APIs.
- The unverifiable push-mode failure must not be cited as proven on-chain fact.
- Private-repo fixes can establish that a class of bug occurred and was addressed by the reporting team; readers cannot independently inspect those commits.
- Example JSON below illustrates research vocabulary only.

## Remaining design decision

What minimal result and error envelope should let an agent decide whether it changed economic state, whether retry is safe, and what it may do next—across both direct-payment and escrow-backed tools?
