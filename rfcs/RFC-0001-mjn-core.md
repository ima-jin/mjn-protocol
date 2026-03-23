> **Changelog — 2026-03-23:** DFOS integration — DFOS is now the L6 identity substrate. The stack diagram updated to reflect this. Key rotation, multifactor key roles (auth/assert/controller), and bilateral attestations are DFOS primitives, not MJN-custom mechanisms. The Attestation primitive now uses DFOS bilateral attestations (countersignatures). See new section: [DFOS Integration](#dfos-integration).

---

# RFC-0001: MJN Protocol Core Specification

| Field | Value |
|-------|-------|
| RFC | 0001 |
| Title | MJN Protocol Core Specification |
| Author | Ryan Veteze (b0b) \<ryan@imajin.ai\> |
| Status | DRAFT |
| Created | 2026-02-25 |
| Updated | 2026-03-23 |
| Repository | github.com/ima-jin/imajin-ai |

---

## Abstract

This document specifies the core architecture of the MJN (iMaJiN Network) protocol — an open application-layer protocol that extends the internet stack with sovereign identity, creative attribution, programmable consent, and automated value settlement.

The protocol is organized around four identity scopes (Actor, Family, Community, Business) and five primitives (Attestation, Communication, Attribution, Settlement, Discovery). Every problem MJN solves is a cell in the matrix formed by their intersection.

MJN is named for 今人 (*ima jin*) — Japanese for "now person." The protocol's foundational unit is the sovereign human presence, not the document, the account, or the platform.

---

## Status of This Document

This is a DRAFT RFC. It is published to establish prior art and invite community review. It does not represent a final specification. Changes are expected.

Discussion: [github.com/ima-jin/mjn-protocol/issues](https://github.com/ima-jin/mjn-protocol/issues)

---

## Motivation

The internet stack answers one question reliably: *did the packet arrive?*

It does not answer:
- Who actually sent this?
- Whose creative work is in this payload?
- Did the creator consent to this use?
- Who receives value when this content is consumed?
- Can the sender be trusted?
- How do I find the person I need?

These questions are currently answered — badly, extractively, inconsistently — by platforms that insert themselves between humans and their own presence on the network. The result is stolen consent, uncompensated creative labor, and identity infrastructure owned by entities with interests misaligned with the humans they serve.

MJN answers all of them at the protocol layer, natively, in every exchange.

---

## Protocol Overview

### Stack Position

```
┌──────────────────────────────────────────────────────────────────────┐
│  MJN                                                                 │
│  attestation · communication · attribution · settlement · discovery  │
├──────────────────────────────────────────────────────────────────────┤
│  DFOS                                                                │
│  identity substrate · content chains · bilateral attestations        │
│  key roles (auth / assert / controller) · relay network              │
├──────────────────────────────────────────────────────────────────────┤
│  HTTP / WebSockets                                                   │
│  transport                                                           │
├──────────────────────────────────────────────────────────────────────┤
│  TCP/IP                                                              │
│  packets                                                             │
└──────────────────────────────────────────────────────────────────────┘
```

MJN does not replace HTTP. It gives HTTP exchanges sovereign meaning.

### JBOS Architecture

MJN is implemented as **JBOS — Just a Bunch Of Services** running on a cryptographic substrate. The architecture has two distinct layers:

```
                    JBOS (Just a Bunch Of Services)
┌──────────────────────────────────────────────────────────┐
│  www · events · chat · learn · market · coffee · links   │  ← Userspace
│  dykil · input · media                                   │     (disposable)
├──────────────────────────────────────────────────────────┤
│  connections · profile · registry                        │  ← Trust + Discovery
├──────────────────────────────────────────────────────────┤
│  auth · pay                                              │  ← Kernel
│  attestations · .fair manifests · settlement             │     (signed chains)
├──────────────────────────────────────────────────────────┤
│  DFOS Proof Chains (L0–L5)                               │  ← Substrate
│  Ed25519 · CID · dag-cbor · countersignatures            │     (cryptographic)
└──────────────────────────────────────────────────────────┘
```

**The kernel** is `auth` + `pay` + the attestation/settlement layer. Services above the kernel cannot do anything meaningful without it — settlement won't process, consent won't validate, attribution chains won't resolve. The kernel does not care which services exist above it.

**Userspace** services are commodity. Swap them, rewrite them, add more. The value is in the cryptographic substrate underneath — signed identity chains that make every action verifiable and every relationship bilateral. Without the chains, each service is a dumb CRUD app. With the chains, they form a sovereign operating system where identity, attribution, trust, and settlement are structural properties.

**The JBOS thesis:** services are commodity, chains are permanent. DFOS provides the identity substrate MJN builds on: content-addressed chains, key role separation, and cryptographically verifiable bilateral attestations. MJN primitives (Attribution, Attestation, Settlement) operate over DFOS; they do not reimplement what DFOS already provides. An MJN request is a request from a verified identity, with attribution declared, consent embedded, and settlement instruction attached.

### The Architecture: Scopes × Primitives

The protocol is organized around two dimensions.

**Four Identity Scopes** describe who is acting:

| Scope | What It Is |
|-------|-----------|
| **Actor** | One DID, one keypair. The atomic unit. Humans, agents, and devices. |
| **Family** | Intimate trust. Shared resources, delegated authority. |
| **Community** | Shared purpose. Trust earned and attested. |
| **Business** | Structured entity. Roles, hierarchy, delegation chains. |

**Five Primitives** describe what they can do:

| Primitive | What It Carries |
|-----------|----------------|
| **Attestation** | Credentials, reputation, endorsement |
| **Communication** | Scoped messaging within and across trust rings |
| **Attribution** | .fair manifests, revenue chains, creative lineage |
| **Settlement** | Payments, fees, declared-intent marketplace |
| **Discovery** | Federated registry, node presence, queryable expertise |

The protocol IS the matrix. Every use case, every interaction, every settlement — is a cell in this grid.

---

## Identity Scopes

MJN does not treat identity as a flat concept. An "account" with permissions attached after the fact is what every identity system in production today provides. MJN encodes the type at the protocol level. A DID is always one of four scopes, each encoding a fundamentally different kind of entity with different trust semantics, governance models, and graph behavior.

Trust relationships, governance models, and privacy boundaries are structural — they cannot be bypassed by application code because they are properties of the identity itself, not policies applied to it.

### Actor — The Atomic Unit

One DID, one keypair. The foundation everything else is built from.

An Actor is a sovereign identity — generated, not issued. Not revocable by any third party. Not tied to any single infrastructure provider. When you generate an Ed25519 keypair, that's your identity. Nobody issued it. Nobody can revoke it. It's cryptographically yours.

```
did:mjn:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK
```

MJN DIDs are compatible with the W3C DID Core specification. They are portable across all implementations.

**Actor subtypes.** The Actor scope covers three kinds of entities — all with the same keypair structure, the same DID format, the same trust graph, the same .fair attribution:

- **HumanActor** — A person. Self-sovereign. The irreducible unit of identity.
- **AgentActor** — An AI agent. Child key derived from a parent human or organizational DID. Authority scope encoded in the key derivation path — an agent key derived with scope `[attribution:collaborator]` cannot produce a valid signature claiming `[attribution:creator]`. The key either has the authority or it does not.
- **DeviceActor** — A physical device. The LED cube responding to verified presence. The sensor reporting environmental data. Same cryptographic structure, same accountability chain.

**DID Document (minimal):**
```json
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:mjn:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK",
  "type": "HumanActor",
  "verificationMethod": [{
    "id": "did:mjn:z6Mk...#keys-1",
    "type": "Ed25519VerificationKey2020",
    "controller": "did:mjn:z6Mk...",
    "publicKeyMultibase": "z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK"
  }],
  "authentication": ["did:mjn:z6Mk...#keys-1"],
  "service": [{
    "id": "did:mjn:z6Mk...#mjn-node",
    "type": "MJNNode",
    "serviceEndpoint": "https://node.example.com/mjn"
  }]
}
```

#### Progressive Trust: Three Standing Levels

Standing on MJN is computed, not assigned. It is a query over attestation history — not a role granted by an administrator.

| Standing | Label | Access | How You Get There |
|----------|-------|--------|-------------------|
| Soft DID | **Visitor** | Email/magic link. Read-only access to public content. | Show up. Provide an email. |
| Hard DID Preliminary | **Resident** | Keypair present. Pod membership. Cannot vouch for others. | Generate a keypair. Get invited by an existing member. |
| Hard DID Established | **Host** | Full standing. Can vouch. Full governance participation. Can issue attestations. | Earn it. Attend events. Get vouched for. Build history. |

Standing is computed from attestation history: event attendance, vouches received, interactions verified, milestones completed — weighted by recency, by the standing of the people who vouched for you, and by the depth of your participation.

**Properties:**
- Self-sovereign: generated, not issued
- Governance: individual autonomy
- Entry: invitation from existing trust graph
- Default visibility: private — you control what others see

### Family — Intimate Trust

The smallest unit of shared identity. Biological or chosen.

- Formed by mutual attestation between Actor DIDs
- Governance: consensus among members
- Entry: invitation + acceptance by existing members
- Default visibility: private interior, shared exterior

Family is the trust boundary where privacy is structurally different. Members share context that no external party can access — custody, finance, healthcare decisions, emergency access. A parent consenting on behalf of a minor carries a custodial consent declaration that encodes the relationship cryptographically. Settlement for family resources flows through multi-signature authorization.

This is not a group chat with special permissions. The protocol knows the difference because the identity type carries it.

### Community — Shared Purpose

Communities of practice. Art collectives, music scenes, mutual aid networks, open-source projects, festival communities. Entities defined by shared practice rather than legal structure.

- Formed by quorum: minimum founding Actor DIDs with demonstrated participation history
- Governance: trust-weighted — contribution history (.fair), activity recency, and attestation weight determine authority
- Entry: formation threshold + demonstrated participation
- Membership: fluid and tiered (governing, active, participant, observer)
- Default visibility: public face, private interior
- No single Actor DID can hold unilateral control — structural anti-capture
- Profit motive structurally excluded — this is not a business, it's a practice

Community DIDs can maintain their own attestation vocabularies — community-specific types for ceremonies, achievements, roles that the protocol's default vocabulary doesn't cover. They can curate declaration namespaces for discovery matching within their domain.

### Business — Structured Entity

Businesses and legal entities. Incorporated, registered, with named founders and optionally a profit motive.

- Formed by declaration from one or more founding Actor DIDs
- Governance: founder-anchored hierarchy with delegated roles
- Entry: vetting + covenant alignment
- Membership: fixed (employees, partners, with scoped permissions)
- Default visibility: public by default

**The founding anchor is non-severable.** A Business DID is permanently and cryptographically linked to its founding Actor DIDs. Negative attestations against the business propagate a standing penalty to the founders. You cannot create a business, behave badly, and walk away.

**The covenant.** Every Business DID is admitted through a conformance gate: a signed, auditable behavioral checklist. Not a values alignment test — a list of specific, observable behaviors that disqualify an entity.

**Soft-loading.** Businesses don't need to join MJN. Their customers build their presence for them through check-ins and transactions. When the business owner claims the soft Business DID, they inherit verified customer count, transaction volume, and reviews.

### Structural Comparison

| Property | Actor | Family | Community | Business |
|----------|-------|--------|-----------|----------|
| Default visibility | Private | Private interior | Public face, private interior | Public |
| Governance | Individual | Consensus | Trust-weighted quorum | Founder-anchored |
| Entry condition | Invitation | Mutual attestation | Quorum + participation threshold | Declaration + covenant |
| Membership | Singular | Intimate, stable | Fluid, tiered | Fixed, role-scoped |
| Profit motive | N/A | N/A | Structurally excluded | Allowed |
| Can hold funds | Yes | Yes (shared) | Yes (quorum-signed) | Yes |

These are not access tiers. They are not permission levels. They are fundamentally different kinds of entities in the world, and the protocol treats them as such.

---

## The Five Primitives

### Primitive 1: Attestation — The Cryptographic Foundation

Every trust-relevant act on MJN is a cryptographically signed record from the moment it occurs. Not a database entry. Not an unsigned log. A signed, typed, timestamped attestation that proves who said what about whom, and when.

This is the foundation everything else builds on. Standing is computed from attestations. Reputation is a query over attestations. Governance weight derives from attestation history.

#### Requirements

- MUST be cryptographically signed by the issuer's DID key (Ed25519)
- MUST reference both issuer and subject DIDs
- MUST carry a type from the controlled vocabulary (extensible by Community DIDs)
- MUST be anchored to a node context
- MUST be verified at ingestion — unsigned or unverifiable attestations are rejected at write
- SHOULD carry an encrypted payload under the subject DID's key for privacy
- MAY carry an unencrypted `payload_hint` for aggregate computation

#### Schema

```sql
CREATE TABLE auth.attestations (
  id            TEXT PRIMARY KEY,           -- att_xxx
  issuer_did    TEXT NOT NULL,              -- who signed it
  subject_did   TEXT NOT NULL,              -- who it's about
  node_context  TEXT NOT NULL,              -- anchors to relationship root
  type          TEXT NOT NULL,              -- controlled vocabulary
  context_id    TEXT,                       -- event/org/interaction DID or ID
  context_type  TEXT,                       -- 'event' | 'org' | 'interaction' | 'system'
  payload       JSONB DEFAULT '{}',         -- ENCRYPTED under subject_did key
  payload_hint  JSONB DEFAULT '{}',         -- unencrypted aggregate for computation
  signature     TEXT NOT NULL,              -- Ed25519 by issuer — verified at write
  issued_at     TIMESTAMPTZ DEFAULT NOW(),
  expires_at    TIMESTAMPTZ,                -- time-decaying flags
  revoked_at    TIMESTAMPTZ,                -- nullable revocation
  issuer_type   TEXT NOT NULL DEFAULT 'human'  -- 'human' | 'agent' | 'system'
);
```

**Privacy by architecture.** The `payload` field is encrypted under the subject DID's public key at write time. The node operator sees aggregate metadata — type, issuer, timestamp, signature — enough to compute standing. They never see the narrative content. That content belongs to the person it's about, portable with them when they leave.

#### Attestation Types

| Type | Issued By | What It Records |
|------|-----------|----------------|
| `event.attendance` | EventDID | Physical presence at verified event |
| `vouch.given` | Established Actor | Sponsors onboarding of new Actor |
| `vouch.received` | System | Issued at vouch acceptance |
| `checkin.verified` | BusinessDID | Physical presence at business location |
| `interaction.verified` | System | Completed meaningful exchange |
| `milestone.completed` | System | Onboarding period milestone |
| `flag.yellow` | Established DID / governance | Soft concern |
| `flag.amber` | Governance body | Formal concern |
| `flag.red` | Governance body | Severe |
| `flag.cleared` | Governance body | Resolution record |
| `org.founding` | System | Non-severable business accountability anchor |
| `org.checkin.soft` | Any Actor | Check-in at unclaimed business location |
| `org.claim.vouch` | Established Actor | Explicit vetting endorsement for business claim |
| `relationship.root` | Node + Actor (bilateral) | Onboarding anchor — both parties sign |

The vocabulary is versioned and extensible. Community DIDs can propose community-specific attestation types.

#### Standing Computation

```
standing(did, scope?) = f(
  positive_attestations    -- weighted by type, issuer_type, and issuer standing
  negative_attestations    -- flags weighted by severity and recency
  recency_weights          -- time-decay function over issued_at
  issuer_standing          -- recursive: issuer's standing affects weight
  node_context             -- optional: scope to a specific community
)
```

Two distinct queries:

- **Community standing**: trust earned within a specific node. Relevant for local governance, access, flag review.
- **Cross-node standing**: aggregate standing across all nodes. Relevant for portable context, inter-node vouching, Community DID formation.

---

### Primitive 2: Communication — Scoped Messaging

Messaging on MJN is scoped by the trust graph. Not filtered by an algorithm. Not routed through a platform that reads your messages. Scoped: who can talk to whom is determined by the structure of relationships in the graph, and the identity types at both ends determine the rules.

#### Requirements

- MUST be signed by the sender's DID key
- MUST respect scope-specific communication rules
- MUST encrypt end-to-end for Actor-to-Actor messages
- SHOULD produce delivery confirmation as an attestation
- Communication history between two parties MUST be a queryable, signed record

#### Per-Scope Communication Rules

| Scope | Communication Properties |
|-------|------------------------|
| Actor | Private by default. Encrypted end-to-end. Sender and recipient control retention. |
| Family | Shared interior channel. Custodial messaging (parent-child). Emergency broadcast. |
| Community | Tiered channels by membership level. Governance communications require quorum visibility. |
| Business | Cannot initiate connections. Can message existing trust-graph connections. All commercial messaging is consent-gated and gas-priced. |

An Actor-to-Actor message within a Family scope carries different privacy guarantees than a Business-to-Actor message in the Discovery scope. The protocol knows the difference because the identity types carry it.

---

### Primitive 3: Attribution — .fair Manifests

Every piece of creative work that moves through MJN carries a `.fair` manifest: a cryptographically signed document embedded in the work itself — not in a platform database — that records the complete chain of human creative labor that produced it.

#### Requirements

- MUST be signed by the owner's DID key (Ed25519)
- MUST declare all contributors and their proportional shares
- MUST declare any prior works this derives from
- MUST declare terms governing consumption (read, train, redistribute, remix)
- MUST declare settlement instructions for each term
- MUST include machine-readable consent declarations
- Shares MUST sum to exactly 1.0
- For multi-contributor manifests, each listed party MUST sign to acknowledge their declared role and share
- Settlement layer MUST reject unsigned or invalid manifests
- SHOULD be embedded in the work artifact where the format permits

#### Schema

```json
{
  "version": "1.0",
  "did": "did:mjn:<author>",
  "created": "<ISO-8601-timestamp>",
  "contributors": [
    {
      "did": "did:mjn:<contributor>",
      "role": "author | editor | source | collaborator | sampled",
      "share": 0.00,
      "signature": "<ed25519-signature>"
    }
  ],
  "derivedFrom": [
    "did:mjn:<prior-work-did>"
  ],
  "terms": {
    "read": {
      "price": 0.00,
      "currency": "MJN | USD | free",
      "consent": "implicit | explicit"
    },
    "train": {
      "price": 0.00,
      "currency": "MJN | USD | prohibited",
      "consent": "explicit"
    },
    "redistribute": {
      "price": 0.00,
      "currency": "MJN | USD | free",
      "conditions": "attribution-required | unrestricted | prohibited"
    },
    "remix": {
      "price": 0.00,
      "currency": "MJN | USD | prohibited",
      "conditions": "attribution-required | unrestricted | prohibited"
    }
  },
  "canonical": "https://example.com/work",
  "signature": {
    "algorithm": "Ed25519",
    "value": "...",
    "publicKeyRef": "did:mjn:z6Mk...#key-1"
  }
}
```

#### Consent Is Embedded

Consent is not a standalone primitive. It is woven through Attribution and Attestation — because consent without attribution is unenforceable, and attribution without consent is theft.

Every .fair manifest includes machine-readable consent declarations: what the creator permits, under what conditions, at what price. Consent is explicit, versioned, and cryptographically signed. It cannot be implied, assumed, or buried in terms of service.

Consent semantics vary by identity scope:
- An **Actor** consents for themselves.
- A **Family** may carry custodial consent — encoding the custodial relationship cryptographically.
- A **Community**'s consent requires quorum attestation from governing members.
- A **Business**'s consent flows through its delegation hierarchy.

#### Attribution Follows the Identity Graph

Attribution chains are typed by identity scope:

- A .fair manifest from an **Actor** credits a person.
- A .fair manifest from a **Community** DID credits a collective — with internal splits governed by the collective's own governance model, weighted by contribution history.
- A .fair manifest from a **Business** credits a structured entity — with employee contributions visible in the chain, flowing through the corporate delegation hierarchy.

When a track gets sampled, the .fair chain settles automatically. When an AI model trains on consented data, attribution flows through the graph to every contributor. The attribution system doesn't need special logic for each identity type — it follows the identity graph.

---

### Primitive 4: Settlement — Value Flows Through the Graph

Every MJN exchange that carries value includes a settlement instruction: who gets paid, in what proportion, through what mechanism, on what trigger.

Settlement follows the identity graph. When a consumer pays for work attributed to a Community DID, the settlement instruction splits according to the collective's .fair manifest. Value flows through the graph to the Actor DIDs who actually did the work. There is no platform in the middle.

#### Requirements

- MUST reference the .fair manifest of the content being consumed
- MUST specify settlement currency (MJN token by default; USD stablecoin implementations permitted)
- MUST execute atomically — full settlement or no settlement, never partial
- MUST distribute to all contributors in the proportions declared in .fair
- MUST be verifiable on-chain
- Settlement failure MUST invalidate the associated consent declaration

#### Schema

```json
{
  "version": "1.0",
  "consumer": "did:mjn:<consuming-node>",
  "content": "did:mjn:<content-did>",
  "amount": 0.00,
  "currency": "MJN",
  "distribution": [
    { "did": "did:mjn:<contributor>", "amount": 0.00, "share": 0.00 }
  ],
  "txHash": "<on-chain-transaction-hash>",
  "timestamp": "<ISO-8601-timestamp>",
  "signature": "<ed25519-signature>"
}
```

#### Per-Scope Settlement Governance

- **Actor** — simple keypair signing
- **Family** — multi-signature authorization
- **Community** — quorum-signed
- **Business** — delegation chain

#### The MJN Token

Reserve-backed utility token. Dual-currency — every transaction settles in fiat OR MJN. Nobody is forced into crypto.

- Mint on fiat deposit, burn on fiat withdrawal. 1:1 reserve backing.
- Solana transaction cost ≈ $0.001. Every .fair split settles in the same transaction.
- Lower fees than Stripe. Instant settlement. Atomic .fair splits.

Token governance, issuance schedule, and economic parameters are defined in RFC-0004 (forthcoming). Token issuance is governed by the MJN Foundation under Swiss FINMA guidelines.

#### The Declared-Intent Marketplace

Your attention is worth $272/year to Meta. You get $0. MJN inverts this.

Your attention profile lives on your node. It never leaves. Companies pay to match your declared interests. You set the price. You keep the revenue. You revoke access anytime.

**Gas-gated reach:**

| Tier | Reach | Gas Cost |
|------|-------|----------|
| Tier 1 | Trust graph connections | Free / near-free |
| Tier 2 | Declared interest pool, matched | Medium gas |
| Tier 3 | Extended reach, opted-in | High gas |

Frequency-scaled gas gates depth. Gas cost to reach the same person scales exponentially with recency. Cluster-aware computation prevents coordination gaming — if multiple Businesses sharing a founding Actor DID alternate messages, the frequency curve applies to the cluster. k-anonymity enforcement at the matching layer prevents inference attacks.

---

### Primitive 5: Discovery — Federated Registry and Portable Context

Discovery on MJN is how entities find each other, prove their history, and make their expertise queryable — all without a centralized directory that becomes a control point.

#### Requirements

- Nodes MUST announce presence to the federated registry
- Node announcements MUST include operator DIDs and identity scopes served
- Registry MUST be queryable by geography, community affiliation, expertise domain
- Trust-gated queries MUST verify the querier's DID and trust relationship
- Exit credentials MUST be signed summaries of already-signed attestation records
- Exit credentials MUST have two layers: public summary and encrypted context

#### Federated Registry

Every node on the MJN network is a sovereign instance. Nodes discover each other through a federated registry — no central server required. Each node announces its presence, its operators, and the scopes of identity it serves.

#### Trust-Gated Presence

A knowledge leader operates a node. Their node carries their body of work, their trust graph, their consent terms, and their inference pricing. Only people in their trust graph can query their presence. The querier's DID is verified. The trust relationship is checked — not just "are you connected?" but "what type of connection, at what trust depth, through which identity scopes?"

The response is signed. The .fair manifest attributes it. Settlement executes. The querier gets an answer from someone they trust. The expert gets compensated.

#### Portable Context on Exit

When an Actor leaves a node, they receive a signed, encrypted exit credential — a portable summary of their attestation history on that node.

**Two layers:**

1. **Public summary layer** — aggregate facts legible to any receiving node: tier reached, attestation count by type, normalized trust score, duration of membership.
2. **Encrypted context layer** — the full attestation history, encrypted under the departing Actor's public key. Presenter-controlled.

**Departure types matter:**

| Departure Type | Context | Trust-Seeding Signal |
|----------------|---------|---------------------|
| Voluntary | Member chose to leave | Neutral — full context preserved |
| Inactivity | Removed for sustained absence | Soft signal — throttled trust growth at new node |
| Community dissolution | Node shut down | Neutral — not the member's choice |
| Behavioral | Removed for conduct violation | Flag category included — no narrative detail |

The exit credential makes leaving cheap. Your reputation follows your keypair, not their database.

---

## The Typed Identity Graph

The same trust graph connects all four scope types. The graph is one data structure, but the scope at the center of the query determines what the graph means.

### Query from an Actor:

```
          Community DID (member of)
              ↑
   Actor → Family (belongs to)
              ↓
          Business DID (employed by)
```

The Actor sees their connections, events, communities, organizations.

### Query from a Community DID:

```
              Actor (governing member, weight: 0.3)
                  ↑
Community DID → Actor (active member, weight: 0.15)
                  ↓
              Actor (participant, weight: 0.05)
                  ↓
              Business DID (sponsor, observer tier)
```

The Community sees its membership tiers, governance weights, contribution history, shared creative output.

### Query from a Family:

```
              Actor (parent, custodial authority)
                  ↑
Family DID → Actor (child, limited authority)
                  ↓
              Business DID (family business)
                  ↓
              Community DID (community membership)
```

The Family sees shared resources, custodial relationships, emergency contacts.

### Query from a Business:

```
              Actor (founder, admin)
                  ↑
Business DID → Actor (employee, scoped)
                  ↓
              Community DID (scene participant, observer)
                  ↓
              Business DID (partner, vendor)
```

The Business sees its hierarchy, delegated authorities, business relationships.

**The same graph, queried from different scopes, yields fundamentally different shapes. This is not a feature. It is the architecture.**

---

## MJN Request Format

An MJN-compliant HTTP request carries protocol primitives as signed headers:

```
POST /mjn/query HTTP/1.1
Host: node.example.com
Content-Type: application/json

X-MJN-DID: did:mjn:z6Mk...
X-MJN-Scope: HumanActor
X-MJN-Attestation: <base64-encoded-attestation-reference>
X-MJN-Settlement: <base64-encoded-settlement-instruction>
X-MJN-Timestamp: 2026-03-12T00:00:00Z
X-MJN-Signature: <ed25519-signature-over-headers>

{
  "query": "...",
  "contentDID": "did:mjn:..."
}
```

A receiving node MUST:
1. Verify the `X-MJN-DID` resolves to a valid DID Document
2. Verify `X-MJN-Scope` matches the DID Document's declared type
3. Verify `X-MJN-Signature` against the sender's public key
4. Verify any referenced .fair manifest is signed and valid
5. Execute the Settlement Instruction before returning a response
6. Return `402 Payment Required` if settlement fails
7. Return `403 Forbidden` if consent terms are not met

---

## DFOS Integration

Imajin has integrated the DFOS protocol as the L6 identity substrate for the MJN stack. This section describes what DFOS provides, what MJN still owns, and how the boundary works.

### What DFOS Provides

**Identity chains.** Every MJN identity (Actor, Family, Community, Business) is backed by a DFOS identity chain — an append-only, content-addressed chain of signed operations. The chain is the canonical identity record. DIDs are operational aliases that resolve to the chain.

**Multifactor key roles.** DFOS separates Ed25519 keys by role:

| Role | Purpose |
|------|---------|
| `auth` | Authentication — proves you are who you claim to be |
| `assert` | Assertion — signs attestations and attribution claims |
| `controller` | Control — rotates keys, manages the identity chain |

Prior to DFOS integration, MJN key rotation was an open question (see RFC-0002). Key rotation is now a DFOS primitive — the `controller` key publishes a rotation operation on the identity chain. Historical signatures under the old `assert` key remain valid because they are anchored to the chain state at the time of signing. MJN implementations MUST NOT reimplement key rotation; they MUST delegate to the DFOS chain.

**Bilateral attestations.** The DFOS attestation model requires countersignatures from both the issuer and the subject. This is the mechanism underlying the `relationship.root` attestation type already defined in the MJN attestation vocabulary — it now has an explicit DFOS primitive backing it. All MJN attestations of significance (identity, trust edges, attribution consent) MUST be bilateral under DFOS. Unilateral attestations are demoted to hints — they can influence standing computation but cannot gate settlement or consent.

**Relay network.** DFOS operates a relay network. MJN nodes that boot as DFOS relays inherit the relay network's federation topology. This replaces the need for a custom MJN federation protocol in the near term.

### What MJN Still Owns

DFOS provides the substrate. MJN provides the semantics layered on top:

- **Identity scopes** (Actor/Family/Community/Business) — DFOS chains are untyped; MJN encodes scope type in the DID Document
- **The five primitives** — Communication, Attribution, Settlement, and Discovery are MJN-layer semantics
- **`.fair` manifests** — content chain genesis on DFOS, but with MJN-defined attribution schema and consent terms
- **The declared-intent marketplace** — gas model, k-anonymity, frequency curves are MJN
- **Standing computation** — the weighted trust score algorithm is MJN; the attestation records it queries are DFOS

### Bilateral Attestations in MJN Context

The existing MJN attestation schema (Primitive 1) remains valid. What changes is the signing requirement for attestations that gate protocol behavior:

**Before DFOS integration:** Attestations were single-signature. The issuer signed; the record was authoritative.

**After DFOS integration:** Attestations of significance MUST carry countersignatures. The subject must sign to acknowledge the attestation before it becomes effective. This matters most for:

- `relationship.root` (onboarding anchor — already bilateral in the schema; now backed by DFOS)
- Attribution claims in `.fair` manifests (contributor countersigns their share)
- Trust edges in the trust graph (see RFC-0005)
- Consent declarations (consumer countersigns the terms they are accepting)

**Schema addition — `counterSignature` field:**

```json
{
  "id": "att_xxx",
  "issuer_did": "did:dfos:<issuer>",
  "subject_did": "did:dfos:<subject>",
  "type": "relationship.root",
  "signature": "<ed25519-sig-by-issuer>",
  "counterSignature": {
    "value": "<ed25519-sig-by-subject>",
    "signedAt": "<ISO-8601-timestamp>"
  },
  "issued_at": "2026-03-23T00:00:00Z"
}
```

Attestations without `counterSignature` for types that require bilateral signing are marked `pending` and do not contribute to standing computation until countersigned.

---

## Security Considerations

**Key compromise:** If a DID private key is compromised, the node's historical signatures remain valid (immutability of the chain). Key rotation procedure is defined in RFC-0002.

**Consent replay attacks:** Consent declarations embedded in .fair manifests MUST include a timestamp and nonce. Receiving nodes MUST reject declarations with timestamps older than 5 minutes or duplicate nonces.

**Settlement atomicity:** Settlement MUST be atomic. Implementations MUST NOT return content before settlement confirms. Race conditions between consent and settlement are a critical vulnerability.

**Sybil resistance:** The trust graph provides Sybil resistance through social attestation. Nodes with no trust graph connections have limited network standing. The vouch chain creates accountability — when you vouch for someone, your standing is partially staked on their behavior. Formal Sybil resistance mechanisms are defined in RFC-0005 (forthcoming).

**Attestation integrity:** Unsigned or unverifiable attestations are rejected at write. The POST endpoint verifies the Ed25519 signature against the issuer DID's public key before storing. Invalid signatures get a 4xx, not a null signature field.

**Agent authority:** AgentActor DIDs derive their authority from key derivation scope. An agent key derived with scope `[attribution:collaborator]` cannot produce a valid signature claiming `[attribution:creator]`. Authority is cryptographically enforced through key derivation, not runtime permission checks.

---

## Open Questions

This RFC invites community input on the following unresolved questions:

1. **Offline settlement:** How should .fair settlement work for works consumed offline or in environments without connectivity?

2. **Legacy content:** What is the migration path for creative work published before MJN? Can .fair manifests be retroactively applied?

3. **Dispute resolution:** What happens when a contributor disputes their declared share in a .fair manifest after publication?

4. **Minimum viable node:** What is the minimum implementation required to be a valid MJN node? How do we prevent the spec from being too heavy for lightweight implementations?

5. **Cross-chain settlement:** MJN token is the default. What are the requirements for compliant USD stablecoin or other currency implementations?

6. **Family DID formation:** What is the minimum attestation threshold for forming a Family DID? How are custodial relationships verified?

7. **Community DID quorum:** What are the minimum founding requirements for Community DIDs? How is "demonstrated participation" measured?

---

## Related RFCs

| RFC | Title | Status |
|-----|-------|--------|
| RFC-0001 | MJN Core Specification (this document) | DRAFT |
| RFC-0002 | did:mjn Method Specification | DRAFT |
| RFC-0003 | Node Registry and Scope Certification | PLANNED |
| RFC-0004 | MJN Token Economics | PLANNED |
| RFC-0005 | Trust Graph and Sybil Resistance | PLANNED |
| RFC-0006 | .fair Manifest Extensions | PLANNED |
| RFC-0007 | Attestation Vocabulary and Extensions | PLANNED |
| RFC-0008 | Communication Protocol and Scoping Rules | PLANNED |
| RFC-0009 | Discovery and Federation Protocol | PLANNED |
| RFC-0010 | Declared-Intent Marketplace and Gas Model | PLANNED |

---

## Reference Implementation

The reference implementation of MJN is maintained by imajin.ai:

- Repository: [github.com/ima-jin/imajin-ai](https://github.com/ima-jin/imajin-ai)
- 14 live services, self-hosted on owned hardware
- 73 registered identities (25 hard DIDs, 48 soft DIDs)
- First demonstration: April 1st, 2026

The reference implementation is one node among many. It does not define the protocol. This RFC defines the protocol.

---

## Governance

MJN is governed by the MJN Foundation, a Swiss *Stiftung* (foundation) currently in formation. The Foundation will hold the protocol specification, operate the RFC process, and govern token issuance under Swiss FINMA guidelines.

The RFC process is modeled on the IETF RFC process. RFCs are proposed by any contributor, reviewed by the Technical Council, and accepted by community consensus. No entity — including imajin.ai and the MJN Foundation — has unilateral authority to modify an accepted RFC.

Foundation incorporation: Q1 2026.

---

## Acknowledgements

The `.fair` attribution protocol was first conceived by Ryan Veteze in May 2025, following examination of the WeR1 distribution algorithm in Johannesburg in April 2025. The trust graph architecture draws from thirty years of community infrastructure building beginning with b0bby's World BBS (1991–1994).

The scopes × primitives architecture was formalized in the v0.3 whitepaper (March 2026), integrating Greg's architectural review series covering attestation data layer, cryptographic trust, .fair attribution integrity, identity tier storage, portable context, Business DID vetting, gas model, and declaration granularity.

The name 今人 (*ima jin*, "now person") gives the protocol its foundational unit: the sovereign human presence, here, now.

---

## Copyright

This RFC is published under CC0 1.0 Universal. No rights reserved. The MJN protocol specification is in the public domain.

Any implementation of this specification is a valid MJN node. No license required. No permission needed.

---

*Ryan Veteze (b0b) · ryan@imajin.ai · 2026-03-23 (DFOS integration update)*
