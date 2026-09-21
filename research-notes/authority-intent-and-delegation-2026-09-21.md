# Authority, Intent, and Delegation - 2026-09-21

Status: Research reference  
Scope: agent authorization, delegated authority, and structural capability limits  
Normative status: non-normative; does not rewrite or approve any RFC

## Purpose

This note collects research inputs for delegated authority in agentic commerce.

Payment authorization and outcome settlement are related but separate questions:

```text
Was the agent authorized to initiate or prepare the transaction?
Should the counterparty receive committed funds yet?
What authority did the agent actually have?
Which dangerous actions were impossible to express?
```

## Working Principles From Current Evidence

1. Do not maximize agent autonomy. Maximize useful autonomy while minimizing economic authority.
2. Delegate execution before delegating destination-setting authority.
3. Make unauthorized economic actions inexpressible, not merely disallowed.
4. Separate exfiltration risk from economic-decision risk.
5. Treat human signing, policy-constrained signing, and autonomous signing as different interaction models.

## Reference Map

| Source | What it contributes | Research caveat |
| --- | --- | --- |
| Mastercard Verifiable Intent | Cryptographic evidence of principal authorization and intent | Authorization complement; does not decide fulfillment or release |
| AP2 / related intent standards | Principal intent, mandates, and delegated authorization | Needs direct comparison with escrow state and release authority |
| Nirium bounded-authority milestone payouts | Automation acts only inside human-defined escrow boundaries | Implementation evidence; not a generic proof for every Level 3 use case |
| Nirium authority by missing parameter | Callable operation lacks arbitrary beneficiary parameter | Strong authority pattern, but only applies when the action genuinely does not need an external payee |
| Trustless Work role graph | Funder, Receiver, Milestone Marker, Approver, Release Signer, Dispute Resolver, Platform | Existing advantage if agent roles remain bounded and explicit |

## Authority Pattern Inventory

| Pattern | Description | Example |
| --- | --- | --- |
| Human-signed preparation | Agent prepares, human signs | Level 1 RFC 0002 boundary |
| Object precommitment | Human creates/funds the economic object; automation operates inside it | Nirium milestone payout flow |
| Missing parameter | Interface cannot express a dangerous destination | Nirium treasury rebalance flow |
| Allowlisted callable surface | Policy or signer can only call selected contracts/functions | Narrow signer over already-created escrow |
| Capability discovery | Tool surface reports what this caller can actually execute | Open RFC 0004 question |

## References

- <https://www.mastercard.com/us/en/news-and-trends/stories/2026/verifiable-intent.html>
- <https://github.com/Trustless-Work/agentic-escrow-research/blob/main/use-cases/nirium-bounded-milestone-payouts.md>
- <https://github.com/Trustless-Work/agentic-escrow-research/blob/main/use-cases/nirium-treasury-missing-parameter.md>
- <https://github.com/Trustless-Work/agentic-escrow-research/blob/main/rfcs/0002-agent-authority-levels.md>

## Analysis To Do

- Map each Trustless Work role to safe Level 1, 2, and 3 agent participation.
- Identify which operations should be structurally unavailable to automated actors.
- Compare intent authorization systems with escrow release authority.
- Decide whether "authority by missing parameter" should become a named RFC 0002 design pattern.
