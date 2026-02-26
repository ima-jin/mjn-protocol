# RFC-0005: Trust Graph and Sybil Resistance

| Field | Value |
|-------|-------|
| RFC | 0005 |
| Title | Trust Graph and Sybil Resistance |
| Author | Ryan Veteze (b0b) \<ryan@imajin.ai\> |
| Status | DRAFT |
| Created | 2026-02-25 |
| Depends on | RFC-0001, RFC-0002, RFC-0003 |

---

## Abstract

This document specifies the MJN Trust Graph — a decentralized social attestation network that provides Sybil resistance, identity verification, and reputation signaling for MJN nodes.

The trust graph is not a social network. It is infrastructure. It exists to answer one question: *Is this node a real person operating in good faith?*

## Motivation

Decentralized identity creates a permissionless participation problem: anyone can create unlimited DIDs at zero cost. Without Sybil resistance, the network becomes vulnerable to:
- Fake identity floods (bots posing as humans)
- Reputation washing (bad actors creating new identities after misbehavior)
- Trust signal dilution (no way to distinguish real humans from automated agents)

The MJN Trust Graph solves this by anchoring identity in **social attestation** — real humans vouching for real humans. Not platforms. Not algorithms. People.

## Specification

### Trust Graph Primitives

#### Attestation

An **attestation** is a signed statement from one DID about another DID.

**Format:**
```json
{
  "version": "1.0",
  "attester": "did:mjn:<attesting-node>",
  "subject": "did:mjn:<subject-node>",
  "attestationType": "identity | nodeType | skill | event",
  "claim": "string describing what is being attested",
  "confidence": 0.0-1.0,
  "timestamp": "<ISO-8601-timestamp>",
  "signature": "<ed25519-signature>"
}
```

**Attestation types:**
- **identity:** "I attest this DID represents a real human I have interacted with"
- **nodeType:** "I attest this DID operates as a `person` node, not a bot or service"
- **skill:** "I attest this person has expertise in [domain]"
- **event:** "I attest this person attended [event] in person"

**TBD:** Full attestation schema, validation rules, revocation mechanisms.

---

#### Trust Score

A **trust score** is a computed value representing the network's confidence that a DID is operated by a real human in good faith.

**Inputs:**
- Number of attestations received
- Quality of attesters (recursive — attestations from high-trust nodes count more)
- Attestation age (recent attestations weighted higher)
- Diversity of attesters (attestations from independent subgraphs count more)
- Behavioral signals (transaction history, `.fair` publication, settlement participation)

**Output:** Score from 0.0 (bot/Sybil) to 1.0 (high-confidence human)

**TBD:** Full trust score algorithm, weight parameters, update frequency.

---

#### Trust Horizon

A **trust horizon** is the maximum graph distance at which attestations are counted.

Example: If Alice trusts Bob, and Bob trusts Carol, Alice may transitively trust Carol (2-hop trust). But if trust horizon = 2, attestations beyond 2 hops don't contribute to Alice's view of the network.

**TBD:** Recommended default trust horizon, user configurability, distance weighting.

---

### Sybil Resistance Mechanisms

#### 1. Social Attestation Bottleneck

Creating a DID is free. Getting attestations from real humans is not.

A Sybil attacker must either:
- Convince real humans to attest fake identities (social engineering, expensive at scale)
- Compromise real human DIDs to issue fake attestations (difficult, detectable)
- Create fake attestations from fake DIDs (low trust score, doesn't propagate)

**Mitigation strength:** Proportional to attestation diversity and attester quality.

**TBD:** Minimum attestation threshold for meaningful trust scores.

---

#### 2. Cost-of-Acquisition Scaling

Each additional fake identity requires new attestations from distinct real humans. The cost scales linearly with attack size.

**TBD:** Economic analysis of attack costs at various scales.

---

#### 3. Behavioral Reputation Signals

Beyond attestations, trust scores incorporate on-chain behavior:
- `.fair` manifest publication history
- Settlement transaction participation
- Time since DID creation (older DIDs weighted higher)
- Activity patterns (bots have different temporal signatures than humans)

**TBD:** Behavioral signal weights, anomaly detection algorithms.

---

#### 4. Stake-Weighted Attestations (Optional)

Attesters MAY stake MJN tokens on their attestations. If the attested node is later flagged for Sybil behavior, staked tokens are slashed.

**Trade-off:** Adds economic weight to attestations, but introduces wealth bias (rich attesters' vouches count more).

**Open question:** Should stake-weighting be protocol-level or application-level?

---

### Privacy Considerations

**Problem:** Public attestations reveal social graphs. This creates:
- Surveillance risks (who trusts whom is publicly visible)
- Targeted attack vectors (adversaries can map trust relationships)
- Social pressure (users may not want to publicly attest or not-attest)

**Solution approaches:**

#### Option 1: Public Attestations (transparency-first)
- All attestations published on-chain or in public registry
- Full transparency, maximum verifiability
- Privacy trade-off: social graph is public

#### Option 2: Zero-Knowledge Attestations (privacy-first)
- Attestations exist but content is cryptographically hidden
- Nodes prove "I have N attestations from high-trust nodes" without revealing which nodes
- Privacy-preserving, but adds complexity and reduces transparency

#### Option 3: Selective Disclosure (hybrid)
- Attestations are private by default
- Attesters choose what to disclose publicly
- Verifiers see aggregated trust scores, not individual attestations

**Community input needed:** Which approach best balances Sybil resistance and privacy?

---

### Trust Graph Queries

Nodes MUST be able to query:
1. **Trust score for a DID:** "What is the network's confidence this is a real human?"
2. **Attestations for a DID:** "Who has vouched for this identity, and what did they claim?"
3. **Trust path between two DIDs:** "How are Alice and Carol connected via mutual attestations?"

**TBD:** Query API specification, indexing requirements, caching strategies.

---

### Attestation Revocation

Attestations MUST be revocable.

**Revocation reasons:**
- Attester discovers they were deceived (subject was a bot)
- Relationship ended (no longer confident in subject's good faith)
- Subject behavior changed (was trustworthy, no longer is)

**Requirements:**
- Revocations MUST be signed by original attester
- Revoked attestations MUST remain in historical record (auditability)
- Trust scores MUST recalculate when attestations are revoked

**TBD:** Revocation format, propagation mechanism, grace periods.

---

## Rationale

### Why not Proof of Work or Proof of Stake for Sybil resistance?

PoW and PoS create wealth and resource barriers. MJN is designed to be accessible to every human, regardless of economic status or access to computational resources.

Social attestation is the only Sybil resistance mechanism that doesn't discriminate by wealth or infrastructure.

### Why not CAPTCHA or biometric verification?

CAPTCHAs are bypassable, annoying, and often exploitative (unpaid labor for AI training). Biometrics create privacy risks and centralized identity databases.

Social attestation is decentralized, privacy-preserving (if designed well), and resistant to automated attacks.

### Why not web-of-trust models like PGP?

PGP's web of trust failed at scale because:
- No automated trust score calculation (manual verification burden)
- No Sybil resistance (easy to create fake keys and self-attest)
- Poor UX (key signing parties, manual trust decisions)

MJN trust graph automates trust score calculation, incorporates economic signals, and provides clear UI for attestation.

---

## Security Considerations

### Collusion Attacks

If a group of Sybil identities attest each other, they can artificially inflate trust scores.

**Mitigation:** Trust score algorithm MUST weight attestation diversity. Tightly connected clusters of mutual attestation should score lower than attestations from independent subgraphs.

**TBD:** Cluster detection algorithms, subgraph diversity metrics.

---

### Attestation Selling

If attestations have economic value (higher trust = more network access), a black market for attestations will emerge.

**Mitigation:**
- Stake slashing for fraudulent attestations
- Reputation damage for attesters whose subjects are later flagged
- Behavioral anomaly detection (users with purchased attestations behave differently than organic users)

**TBD:** Marketplace detection, penalty mechanisms.

---

### Privacy Leakage via Graph Analysis

Even if individual attestations are hidden, graph structure analysis can reveal:
- Community boundaries (who trusts whom)
- Influence networks (central nodes)
- Targeted attack vectors (compromise high-trust nodes)

**Mitigation:** Zero-knowledge proofs, selective disclosure, aggregation layers that obscure individual edges.

**TBD:** Full privacy-preserving query mechanisms.

---

### Trust Score Manipulation

Adversaries may attempt to game the trust score algorithm by:
- Creating legitimate-looking activity patterns
- Slowly accumulating real attestations over time
- Operating in good faith initially, then defecting (reputation laundering)

**Mitigation:**
- Time-weighting (recent behavior matters more)
- Anomaly detection (sudden behavior changes trigger re-evaluation)
- Community flagging mechanisms

**TBD:** Reputation laundering detection algorithms.

---

## Open Questions

1. **Privacy vs transparency tradeoff:** Should attestations be public, zero-knowledge, or user-configurable?

2. **Minimum attestation threshold:** How many attestations are required before a DID has a meaningful trust score?

3. **Trust horizon default:** What's the right default for maximum trust propagation distance?

4. **Stake-weighting:** Should economic stake amplify attestation weight, or does this create plutocracy?

5. **Cross-cultural trust models:** Do attestation norms vary by culture? Should the protocol accommodate different trust models?

6. **Bad actor recovery:** If a DID is flagged as Sybil or bad actor, can they recover their reputation? Or is it permanent?

7. **Bootstrap problem:** In the early network, nobody has attestations. How do we cold-start the trust graph?

8. **AI agent attestation:** Should AI agents be attestable as "trustworthy AI" or does attestation only apply to humans?

## Related RFCs

| RFC | Title | Status |
|-----|-------|--------|
| RFC-0001 | MJN Core Specification | DRAFT |
| RFC-0002 | did:mjn Method Specification | DRAFT |
| RFC-0003 | Node Type Registry and Certification | DRAFT |
| RFC-0005 | Trust Graph and Sybil Resistance (this document) | DRAFT |

## Reference Implementation

**TBD:** Link to reference implementation when available.

Reference implementation will include:
- Attestation creation and signing utilities
- Trust score calculation engine
- Trust graph query API
- Revocation tooling
- Privacy-preserving query mechanisms

## Copyright

CC0 1.0 Universal. No rights reserved.

---

**Community input CRITICAL on:**
- Privacy vs transparency tradeoff (public vs ZK vs hybrid attestations)
- Trust score algorithm design
- Stake-weighting pros/cons
- Bootstrap strategies for early network
- Cultural variation in trust models

Discussion: [github.com/ima-jin/mjn-protocol/issues](https://github.com/ima-jin/mjn-protocol/issues)

---

*Ryan Veteze (b0b) · ryan@imajin.ai · 2026-02-25*
