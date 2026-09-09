# Conditional Settlement for the Agentic Economy

Version: v0 draft  
Status: living whitepaper  
Maintainers: Trustless Work community

## Abstract

AI agents are beginning to act as economic operators. They can search, compare, negotiate, order, hire, consume services, and coordinate work. As agents become more capable, they will need payment infrastructure.

Stablecoins are a natural settlement layer for this environment because they are programmable, global, and machine-accessible. But payment rails alone are not enough. The agentic economy also needs programmable trust boundaries.

This paper explores agentic escrow: conditional settlement infrastructure that allows humans, agents, merchants, and autonomous services to commit funds without making payment final until defined conditions are met.

The central claim is simple:

> Agent wallets answer whether an agent can pay. Escrow answers when a payment should become final.

## 1. The Emerging Agentic Economy

The first wave of AI agents has focused on research, writing, coding, browsing, and workflow automation. The next wave will increasingly touch economic activity:

- purchasing goods;
- booking services;
- consuming APIs and compute;
- hiring contractors or other agents;
- managing recurring software procurement;
- matching grants, bounties, and work;
- coordinating business payments.

These workflows will not always be fully autonomous. Many will combine human intent, agent execution, policy constraints, merchant responses, and third-party evidence.

The payment layer must therefore support partial autonomy.

## 2. Why Agent Wallets Are Not Enough

A common design is to give an agent a wallet with a balance or spending cap. This is useful, but incomplete.

A spending cap can limit amount, but it does not guarantee:

- the purchase matches the user's intent;
- the merchant will fulfill;
- fulfillment evidence is valid;
- the counterparty is the right counterparty;
- the agent did not misunderstand the task;
- the agent was not manipulated;
- the payment can be disputed or recovered.

Direct payment is best when the transaction is immediate, low-risk, and finality is acceptable. Many agentic transactions will not fit that profile.

## 3. Escrow as a Conditional Trust Layer

Escrow creates a protected commitment:

```text
intent -> terms -> committed funds -> evidence -> approval -> release
```

This separates payment commitment from payment finality.

For agentic workflows, this matters because an agent may be trusted to discover, negotiate, and prepare a transaction without being trusted to irreversibly transfer value.

Escrow can encode:

- who funds;
- who receives;
- who marks progress;
- who approves;
- who releases;
- who resolves disputes;
- what evidence is expected;
- what fallback path exists.

## 4. The Trustless Work Hypothesis

Trustless Work provides stablecoin escrow infrastructure. The research hypothesis is that this infrastructure can support agentic commerce by giving agents machine-readable workflows with bounded authority.

The initial model should compose existing escrow primitives through APIs, SDKs, MCP-style tools, and policy layers. It should not require changing audit-bound contracts before the need is validated.

## 5. Autonomy Levels

Agentic payments should be modeled as levels of authority:

| Level | Description | Example |
| --- | --- | --- |
| 0 | Human performs every action | Human creates, funds, approves, and releases |
| 1 | Agent prepares, human signs | Agent drafts escrow terms; human signs all transactions |
| 2 | Agent orchestrates, human funds | Agent creates order and escrow; human deposits funds |
| 3 | Agent acts within policy | Agent can approve or release within predefined limits |
| 4 | Agent-to-agent settlement | Agents transact with bounded rules and dispute/fallback mechanisms |

Early implementations should begin at levels 1 or 2.

## 6. Baseline Agentic Escrow Flow

```text
Human gives purchasing intent to Buyer Agent
Buyer Agent discovers or negotiates offer
Seller Agent or merchant confirms terms
Escrow is created for the order
Human or policy-controlled wallet funds escrow
Merchant fulfills
Evidence is submitted
Approver verifies or disputes
Release signer settles funds
Final state is inspectable by both sides
```

This flow keeps the human in control of the economic commitment while allowing agents to automate discovery, negotiation, tracking, and evidence handling.

## 7. Agent-Facing Interface

Agents need a different interface than human developers.

A useful agent-facing interface should be:

- typed;
- deterministic;
- state-aware;
- idempotent where possible;
- explicit about required signer and role;
- explicit about unsigned, signed, and submitted transaction states;
- machine-readable on errors and next actions;
- easy to inspect before committing funds.

Candidate operations:

- `create_protected_payment`
- `get_escrow`
- `fund_escrow`
- `submit_fulfillment`
- `approve_fulfillment`
- `raise_dispute`
- `release_payment`
- `cancel_or_refund_if_permitted`

These are product-level concepts, not final API names.

## 8. x402 and Escrow

x402-style payment flows are relevant for machine-native payments. They may be especially strong for immediate, low-risk payments for APIs, content, compute, or digital services.

Escrow is relevant when payment finality should depend on fulfillment, approval, verification, delivery, or recourse.

The likely composition is:

```text
x402 -> immediate access fee or payment request
escrow -> conditional principal or protected commitment
```

The research question is not whether escrow replaces x402. The better question is when direct machine payment is sufficient and when conditional settlement is required.

## 9. Safety and Failure Modes

The research must address:

- agent identity and key custody;
- maximum amount, daily, and counterparty limits;
- delegated operations;
- separation of approval and release;
- order-to-escrow binding;
- replay and idempotency protection;
- malicious merchant agents;
- manipulated buyer agents;
- ambiguous fulfillment evidence;
- cancellation and refund semantics;
- dispute resolution;
- user recovery if an agent disappears;
- inspection of open commitments.

No serious agentic payment architecture can ignore these questions.

## 10. Open Research Agenda

1. What is the minimal useful agentic escrow flow?
2. Which operations can be safely delegated to an agent?
3. What must remain human-approved in early deployments?
4. How should an invoice or order bind to an escrow?
5. What evidence is enough for release?
6. How should an agent inspect escrow state?
7. What error model best supports autonomous recovery?
8. Which use cases need escrow instead of direct payment?
9. Where does x402 complement escrow?
10. What reference implementation would teach the most?

## 11. Conclusion

The agentic economy should not be built only around agents that can pay. It should be built around agents that can make commitments safely.

Escrow gives agentic systems a way to coordinate payment, evidence, approval, release, and recourse without forcing every transaction into immediate finality.

Trustless Work's opportunity is to help define the conditional settlement layer for this new economy.
