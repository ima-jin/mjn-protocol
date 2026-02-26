# RFC-0004: MJN Token Economics

| Field | Value |
|-------|-------|
| RFC | 0004 |
| Title | MJN Token Economics |
| Author | Ryan Veteze (b0b) \<ryan@imajin.ai\> |
| Status | DRAFT |
| Created | 2026-02-25 |
| Depends on | RFC-0001 |

---

## Abstract

This document specifies the economic design of the MJN token — the native settlement currency of the MJN protocol. It defines token issuance, distribution, governance, velocity mechanics, and regulatory compliance under Swiss FINMA guidelines.

The MJN token is not a speculative asset. It is settlement infrastructure. It is the mechanism by which value flows through attribution chains.

## Motivation

RFC-0001 establishes that every MJN exchange involving value MUST include settlement, and that settlement defaults to MJN tokens. This RFC defines what the token is, how it's issued, who governs it, and how its economics are designed to support protocol adoption without creating speculative bubbles.

**Key design principle:** Token economics should optimize for network usage (velocity), not speculative holding (store of value). The goal is a stable, liquid settlement layer — not a volatile trading instrument.

## Specification

### Token Fundamentals

**Token name:** MJN
**Token symbol:** MJN
**Blockchain:** **TBD** — Solana, Ethereum L2, or purpose-built chain
**Token standard:** **TBD** — SPL token, ERC-20, or custom
**Decimals:** 6 (1 MJN = 1,000,000 micro-MJN)

**TBD:** Final blockchain selection based on transaction cost, throughput, and settlement finality requirements.

---

### Issuance Schedule

**Total supply cap:** **TBD** — Fixed cap vs uncapped with burn mechanisms

**Initial issuance allocation:**

| Allocation | % | Vesting | Purpose |
|------------|---|---------|---------|
| MJN Foundation Treasury | TBD | N/A | Protocol development, grants, ecosystem growth |
| Early Contributors | TBD | TBD years | Core protocol developers, RFC authors |
| Network Incentives | TBD | Released via usage | Node operator rewards, user acquisition |
| Public Distribution | TBD | TBD | Community access, liquidity bootstrapping |
| Strategic Reserve | TBD | Locked | Emergency reserves, market stability |

**Open question:** Should there be ongoing issuance to reward node operators, or should all tokens be issued upfront with distribution via protocol usage?

---

### Velocity Mechanics

**Core thesis:** Token value correlates with transaction volume, not speculative holding.

Every `.fair` contract execution, inference fee, consent-triggered payment settles in MJN by default. Token velocity = total economic activity flowing through MJN settlement.

**Mechanisms to encourage velocity:**
- **TBD:** Staking rewards for node operators based on settlement volume processed
- **TBD:** Fee rebates for high-velocity nodes
- **TBD:** Burn mechanisms on settlement transactions to create deflationary pressure as usage grows

**Anti-speculation mechanisms:**
- **TBD:** Transfer restrictions or holding penalties to discourage pure trading
- **TBD:** Yield structures that reward network participation over passive holding

**Open question:** How do we balance liquidity (needed for settlement) with anti-speculation (to prevent bubble dynamics)?

---

### Settlement Pricing

**Question:** Should settlement prices be denominated in MJN tokens or fiat-pegged?

**Option 1: MJN-denominated pricing**
- `.fair` manifests declare prices in MJN tokens (e.g., "0.50 MJN per training use")
- Pros: Pure protocol settlement, no fiat dependency
- Cons: Price volatility creates uncertainty for creators

**Option 2: Fiat-pegged pricing with MJN settlement**
- `.fair` manifests declare prices in USD, settled in equivalent MJN at current rate
- Pros: Price stability for creators
- Cons: Requires oracle infrastructure, introduces external dependency

**Option 3: Hybrid**
- Creators choose denomination (MJN or USD)
- Settlement always executes in MJN tokens
- Oracle conversion only for USD-denominated contracts

**Community input needed:** Which approach best balances stability and decentralization?

---

### Governance

**Token holder rights:**
- **TBD:** Do token holders have governance rights over the protocol?
- **TBD:** Should there be a token-holder DAO alongside the Foundation?

**Foundation control:**
- MJN Foundation governs token treasury, issuance schedule, emergency interventions
- Foundation MUST publish quarterly transparency reports on token usage
- Foundation MUST NOT engage in market manipulation or speculative trading

**Open question:** What's the right balance between Foundation stewardship and community governance?

---

### Regulatory Compliance (Swiss FINMA)

The MJN Foundation is incorporated in Switzerland to operate under FINMA's regulatory framework.

**Token classification:**
- **Not a security:** MJN tokens are utility tokens used for settlement, not investment contracts
- **Not a payment token:** MJN tokens are protocol infrastructure, not a currency replacement
- **Settlement token:** MJN tokens are clearinghouse instruments for attribution-based value distribution

**FINMA compliance requirements:**
- **TBD:** AML/KYC requirements for large transactions
- **TBD:** Stablecoin reserve requirements if fiat-pegging is implemented
- **TBD:** Consumer protection disclosures

**Regulatory risk mitigation:**
- Swiss jurisdiction chosen for regulatory clarity and neutrality
- Foundation structure (vs DAO) provides clear legal accountability
- Transparent operation and public auditing

---

### Cross-Chain Settlement

**Requirement from RFC-0001:** MJN token is the default settlement currency, but compliant USD stablecoin or other currency implementations are permitted.

**Requirements for alternative settlement currencies:**
- MUST implement atomic settlement (no partial payments)
- MUST distribute to all `.fair` contributors in declared proportions
- MUST be verifiable on-chain
- MUST publish conversion rates and fee structures

**TBD:** Certification process for alternative settlement currencies, bridge mechanisms, cross-chain atomic swaps.

---

### Economic Projections

From the whitepaper, projected token velocity:

| Scenario | Annual Token Velocity | Timeline |
|----------|----------------------|----------|
| 1% of digital identity market settles via MJN | ~$2B | 2030 |
| AI training data market at full consent model | ~$10-50B | 2028-2030 |
| 1% of global digital economy flows through MJN | ~$160B | 2032-2035 |
| MJN becomes primary internet presence layer | ~$1-5T | 2035+ |

**Honest SAM (2026-2027):** AI training consent market, platform identity fees, node operator settlement — **$50-100M** annual velocity target.

**These are not token price projections.** They are transaction volume projections. Token price is a function of velocity, float, and market dynamics.

---

## Rationale

### Why not use an existing token (ETH, SOL, USDC)?

MJN settlement requires attribution-aware distribution. Generic tokens can't natively split payments to multiple contributors based on `.fair` manifest proportions. MJN tokens are designed for this use case.

### Why not make it a stablecoin?

Stablecoins require reserves and regulatory overhead. MJN is settlement infrastructure, not a fiat replacement. Fiat-pegging (if implemented) should be at the pricing layer, not the token layer.

### Why token-based settlement instead of direct fiat?

- **Programmability:** MJN settlement executes automatically via smart contracts
- **Attribution-native:** Token contracts can encode `.fair` distribution logic
- **Borderless:** No forex conversion, no banking intermediaries
- **Speed:** Crypto settlement is faster than ACH/wire transfers

## Security Considerations

### Token Supply Manipulation

If the Foundation or large holders can manipulate token supply, settlement prices become unreliable.

**Mitigation:** Issuance schedule MUST be transparent and programmatically enforced. Foundation treasury movements MUST be publicly auditable.

### Oracle Attacks (if fiat-pegged)

If settlement uses fiat-pegged pricing, oracle manipulation could drain creator payments.

**Mitigation:** Multi-oracle consensus, outlier detection, rate-limit changes.

**TBD:** Full oracle security requirements.

### Settlement Front-Running

If settlement transactions are publicly visible before execution, attackers could front-run them.

**Mitigation:** Atomic settlement with private mempool or commit-reveal schemes.

**TBD:** Full front-running prevention mechanisms.

## Open Questions

1. **Blockchain selection:** Solana (speed), Ethereum L2 (ecosystem), or purpose-built chain (sovereignty)?

2. **Supply cap:** Fixed supply (Bitcoin model) or dynamic supply with burns (Ethereum EIP-1559 model)?

3. **Staking rewards:** Should node operators earn newly issued tokens, or only transaction fees?

4. **Fiat pegging:** MJN-denominated pricing, fiat-pegged pricing, or hybrid creator choice?

5. **Governance rights:** Should token holders vote on protocol changes, or is Foundation stewardship sufficient?

6. **Speculator deterrence:** How do we design for velocity without killing liquidity?

7. **Treasury transparency:** Should all Foundation wallet addresses be public, or only aggregated reports?

8. **Cross-chain bridges:** If MJN settles on multiple chains, how do we prevent fragmentation?

## Related RFCs

| RFC | Title | Status |
|-----|-------|--------|
| RFC-0001 | MJN Core Specification | DRAFT |
| RFC-0004 | MJN Token Economics (this document) | DRAFT |
| RFC-0005 | Trust Graph and Sybil Resistance | DRAFT |

## Reference Implementation

**TBD:** Link to token contracts when available.

Reference implementation will include:
- Token smart contracts
- Settlement execution engine
- Oracle integration (if fiat-pegged)
- Treasury transparency dashboard

## Copyright

CC0 1.0 Universal. No rights reserved.

---

**Community input CRITICAL on:**
- Blockchain selection (Solana vs Ethereum L2 vs custom)
- Pricing model (MJN-denominated vs fiat-pegged vs hybrid)
- Token holder governance rights
- Speculation deterrence mechanisms
- Issuance schedule and allocation

Discussion: [github.com/ima-jin/mjn-protocol/issues](https://github.com/ima-jin/mjn-protocol/issues)

---

*Ryan Veteze (b0b) · ryan@imajin.ai · 2026-02-25*
