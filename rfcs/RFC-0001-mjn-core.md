# RFC-0001: MJN Protocol Core Specification

| Field | Value |
|-------|-------|
| RFC | 0001 |
| Title | MJN Protocol Core Specification |
| Author | Ryan Veteze (b0b) \<ryan@imajin.ai\> |
| Status | DRAFT |
| Created | 2026-02-25 |
| Repository | github.com/ima-jin/mjn-protocol |

---

## Abstract

This document specifies the core primitives of the MJN (iMaJiN Network) protocol — an open application-layer protocol that extends the internet stack with four capabilities HTTP was never designed to carry: sovereign identity, creative attribution, programmable consent, and automated value settlement.

MJN is named for 今人 (*ima jin*) — Japanese for "now person." The protocol's foundational unit is the sovereign human presence, not the document, the account, or the platform.

---

## Status of This Document

This is a DRAFT RFC. It is published to establish prior art and invite community review. It does not represent a final specification. Changes are expected.

Discussion: [github.com/ima-jin/mjn-protocol/discussions](https://github.com/ima-jin/mjn-protocol/discussions)

---

## Motivation

The internet stack answers one question reliably: *did the packet arrive?*

It does not answer:
- Who actually sent this?
- Whose creative work is in this payload?
- Did the creator consent to this use?
- Who receives value when this content is consumed?

These questions are currently answered — badly, extractively, inconsistently — by platforms that insert themselves between humans and their own presence on the network. The result is stolen consent, uncompensated creative labor, and identity infrastructure owned by entities with interests misaligned with the humans they serve.

MJN answers all four questions at the protocol layer, natively, in every exchange.

---

## Protocol Overview

### Stack Position

```
┌─────────────────────────────────────────────────┐
│  MJN                                            │
│  identity · attribution · consent · settlement  │
├─────────────────────────────────────────────────┤
│  HTTP / WebSockets                              │
│  transport                                      │
├─────────────────────────────────────────────────┤
│  TCP/IP                                         │
│  packets                                        │
└─────────────────────────────────────────────────┘
```

MJN does not replace HTTP. It gives HTTP exchanges sovereign meaning. An MJN request is a request from a verified identity, with attribution declared, consent explicit, and settlement instruction attached.

---

## Core Primitives

### Primitive 1: DID — Sovereign Identity

Every MJN node is identified by a Decentralized Identifier (DID).

**Requirements:**
- MUST conform to W3C DID Core specification v1.0
- MUST use the `did:mjn` method (defined in RFC-0002)
- MUST be cryptographically self-sovereign — not issuable or revocable by any third party
- MUST be portable across MJN implementations
- SHOULD resolve to a DID Document containing the node's public key(s) and service endpoints

**Format:**
```
did:mjn:<multibase-encoded-public-key>
```

**Example:**
```
did:mjn:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK
```

**DID Document (minimal):**
```json
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:mjn:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK",
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

---

### Primitive 2: .fair — Attribution Manifest

Every piece of creative work published through MJN MUST carry a `.fair` manifest.

The manifest is a cryptographically signed JSON document embedded in or attached to the work. It travels with the work. It is immutable after signing. It is owned by nobody. It is verifiable by anyone.

**Requirements:**
- MUST be signed by the author's DID key
- MUST declare all contributors and their proportional shares
- MUST declare any prior works this derives from
- MUST declare terms governing consumption (read, train, redistribute, remix)
- MUST declare settlement instructions for each term
- SHOULD be embedded in the work artifact where the format permits
- SHOULD declare a canonical location for the unsigned source

**Schema:**
```json
{
  "version": "1.0",
  "did": "did:mjn:<author>",
  "created": "<ISO-8601-timestamp>",
  "contributors": [
    {
      "did": "did:mjn:<contributor>",
      "role": "author | editor | source | collaborator | sampled",
      "share": 0.00
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
  "signature": "<ed25519-signature>"
}
```

**Share validation:** All contributor shares MUST sum to exactly 1.0.

**Example — article with editor and source:**
```json
{
  "version": "1.0",
  "did": "did:mjn:z6Mk...",
  "created": "2026-02-25T00:00:00Z",
  "contributors": [
    { "did": "did:mjn:z6Mk...", "role": "author", "share": 0.85 },
    { "did": "did:mjn:z6Ab...", "role": "editor", "share": 0.10 },
    { "did": "did:mjn:z6Cd...", "role": "source", "share": 0.05 }
  ],
  "derivedFrom": [],
  "terms": {
    "read":         { "price": 0.08,  "currency": "MJN", "consent": "implicit" },
    "train":        { "price": 0.50,  "currency": "MJN", "consent": "explicit" },
    "redistribute": { "price": 0.00,  "currency": "free", "conditions": "attribution-required" },
    "remix":        { "price": "prohibited" }
  },
  "canonical": "https://imajin.ai/articles/example",
  "signature": "..."
}
```

---

### Primitive 3: Consent Declaration

Every MJN exchange that involves creative content MUST include a Consent Declaration from the consuming node.

The Consent Declaration is a signed statement of what the consumer intends to do with the content of this exchange. It binds the consumer to the terms declared in the `.fair` manifest. It is machine-readable. It executes automatically via the settlement layer.

**Requirements:**
- MUST reference the `.fair` manifest of the content being consumed
- MUST declare the intended use (read | train | redistribute | remix)
- MUST be signed by the consuming node's DID key
- MUST be transmitted before or with the consumption request
- A consuming node that trains on content without a valid Consent Declaration for `train` is in protocol violation

**Schema:**
```json
{
  "version": "1.0",
  "consumer": "did:mjn:<consuming-node>",
  "content": "did:mjn:<content-did>",
  "intendedUse": "read | train | redistribute | remix",
  "timestamp": "<ISO-8601-timestamp>",
  "signature": "<ed25519-signature>"
}
```

**Note on the distillation problem:** The February 2026 distillation attacks on Anthropic and OpenAI by Chinese AI laboratories represent the precise failure mode this primitive addresses. Under MJN, a Consent Declaration for `train` triggers automatic settlement per the `.fair` manifest terms. The attack becomes a market transaction. The creator gets paid. The lab gets the data. No fraud required.

---

### Primitive 4: Settlement Instruction

Every MJN exchange that involves value MUST include a Settlement Instruction.

Settlement executes automatically when a Consent Declaration is accepted. There is no platform in the middle. Value flows from the consuming node to the contributing nodes in the proportions declared in the `.fair` manifest.

**Requirements:**
- MUST reference the `.fair` manifest of the content being consumed
- MUST specify settlement currency (MJN token by default; USD stablecoin implementations permitted)
- MUST execute atomically — full settlement or no settlement, never partial
- MUST distribute to all contributors in the proportions declared in `.fair`
- MUST be verifiable on-chain
- Settlement failure MUST invalidate the Consent Declaration

**Schema:**
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

---

## MJN Request Format

An MJN-compliant HTTP request carries protocol primitives as signed headers:

```
POST /mjn/query HTTP/1.1
Host: node.example.com
Content-Type: application/json

X-MJN-DID: did:mjn:z6Mk...
X-MJN-Consent: <base64-encoded-consent-declaration>
X-MJN-Settlement: <base64-encoded-settlement-instruction>
X-MJN-Timestamp: 2026-02-25T00:00:00Z
X-MJN-Signature: <ed25519-signature-over-headers>

{
  "query": "...",
  "contentDID": "did:mjn:..."
}
```

A receiving node MUST:
1. Verify the `X-MJN-DID` resolves to a valid DID Document
2. Verify `X-MJN-Signature` against the sender's public key
3. Verify the Consent Declaration references a valid `.fair` manifest
4. Execute the Settlement Instruction before returning a response
5. Return `402 Payment Required` if settlement fails
6. Return `403 Forbidden` if consent terms are not met

---

## The MJN Token

The MJN token is the native settlement currency of the protocol.

It is not a speculative instrument. It is the mechanism by which value moves through attribution chains. Every `.fair` contract execution, every inference fee, every consent-triggered payment settles in MJN tokens by default.

Token governance, issuance schedule, and economic parameters are defined in RFC-0004 .

Token issuance is governed by the MJN Foundation under Swiss FINMA regulatory framework.

---

## Node Types

MJN defines four node types in the operator registry:

| Type | Description | Example |
|------|-------------|---------|
| `person` | Individual now person node | Personal identity + catalogue |
| `community` | Multi-person trust graph node | Event, family, group |
| `operator` | Service node built on MJN | imajin.ai, Substack integration |
| `infrastructure` | Protocol infrastructure node | Foundation nodes, relay nodes |

Node type certification is defined in RFC-0003 .

---

## Security Considerations

**Key compromise:** If a DID private key is compromised, the node's historical signatures remain valid (immutability of the chain). Key rotation procedure is defined in RFC-0002.

**Consent replay attacks:** Consent Declarations MUST include a timestamp and nonce. Receiving nodes MUST reject Declarations with timestamps older than 5 minutes or duplicate nonces.

**Settlement atomicity:** Settlement MUST be atomic. Implementations MUST NOT return content before settlement confirms. Race conditions between consent and settlement are a critical vulnerability.

**Sybil resistance:** The trust graph provides Sybil resistance through social attestation. Nodes with no trust graph connections have limited network standing. Formal Sybil resistance mechanisms are defined in RFC-0005 .

---

## Open Questions

This RFC invites community input on the following unresolved questions:

1. **Offline settlement:** How should `.fair` settlement work for works consumed offline or in environments without connectivity?

2. **Legacy content:** What is the migration path for creative work published before MJN? Can `.fair` manifests be retroactively applied?

3. **Dispute resolution:** What happens when a contributor disputes their declared share in a `.fair` manifest after publication?

4. **AI-generated content:** What DID structure governs content generated by AI nodes? Who holds the key? How is the training attribution chain handled?

5. **Minimum viable node:** What is the minimum implementation required to be a valid MJN node? How do we prevent the spec from being too heavy for lightweight implementations?

6. **Cross-chain settlement:** MJN token is the default. What are the requirements for compliant USD stablecoin or other currency implementations?

---

## Related RFCs

| RFC | Title | Status |
|-----|-------|--------|
| [RFC-0001](RFC-0001-mjn-core.md) | MJN Core Specification (this document) | DRAFT |
| [RFC-0002](RFC-0002-did-mjn-method.md) | did:mjn Method Specification | DRAFT |
| [RFC-0003](RFC-0003-node-registry.md) | Node Type Registry and Certification | DRAFT |
| [RFC-0004](RFC-0004-token-economics.md) | MJN Token Economics | DRAFT |
| [RFC-0005](RFC-0005-trust-graph.md) | Trust Graph and Sybil Resistance | DRAFT |
| [RFC-0006](RFC-0006-fair-extensions.md) | .fair Manifest Extensions | DRAFT |
| [RFC-0007](RFC-0007-government-identity.md) | Government Identity Attestation Layer | DRAFT |

---

## Reference Implementation

The reference implementation of MJN is maintained by imajin.ai:

- Repository: [github.com/ima-jin/imajin-ai](https://github.com/ima-jin/imajin-ai)
- Live services: auth.imajin.ai · profile.imajin.ai · registry.imajin.ai · chat.imajin.ai
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

The name 今人 (*ima jin*, "now person") gives the protocol its foundational unit: the sovereign human presence, here, now.

---

## Copyright

This RFC is published under CC0 1.0 Universal. No rights reserved. The MJN protocol specification is in the public domain.

Any implementation of this specification is a valid MJN node. No license required. No permission needed.

---

*Ryan Veteze (b0b) · ryan@imajin.ai · 2026-02-25*
