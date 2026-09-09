# RFC 0002: Agent Authority Levels

Status: Draft

## Summary

This RFC defines a staged model for agent authority in escrow-protected payments.

Agentic payments should not be treated as fully manual or fully autonomous. Authority exists on a spectrum.

## Authority Levels

| Level | Name | Description |
| --- | --- | --- |
| 0 | Manual | Human creates, funds, approves, and releases |
| 1 | Prepared by agent | Agent prepares terms and transactions; human signs |
| 2 | Agent-orchestrated | Agent creates or coordinates escrow; human funds |
| 3 | Policy-constrained | Agent can perform selected actions within explicit policy limits |
| 4 | Autonomous settlement | Agents transact with defined dispute, fallback, and recovery mechanisms |

## Recommended Initial Boundary

The first public experiments should target Level 1 or Level 2.

This preserves a meaningful human-controlled economic boundary while still allowing agents to automate:

- discovery;
- comparison;
- negotiation;
- order generation;
- escrow setup;
- state monitoring;
- evidence collection;
- release recommendations.

## Delegation Matrix

| Operation | Level 1 | Level 2 | Level 3 |
| --- | --- | --- | --- |
| Search offers | Agent | Agent | Agent |
| Negotiate terms | Agent drafts | Agent drafts | Agent within policy |
| Create escrow | Human signs | Agent may initiate | Agent within policy |
| Fund escrow | Human signs | Human signs | Agent within amount policy |
| Submit evidence | Agent may submit | Agent may submit | Agent may submit |
| Approve fulfillment | Human signs | Human or policy | Agent if permitted |
| Release funds | Human signs | Human or role wallet | Agent if permitted |
| Raise dispute | Human or agent with reason | Human or bounded agent | Agent within policy |

## Safety Requirements

Any delegated economic operation should define:

- max amount;
- max frequency;
- allowed counterparties;
- allowed asset;
- expiration;
- required evidence;
- revocation path;
- human override path.

## Open Questions

- Which operation is the first safe Level 3 candidate?
- Should approval and release ever be controlled by the same agent?
- Should merchant reputation affect authority limits?
- How should failed or ambiguous evidence downgrade autonomy?
