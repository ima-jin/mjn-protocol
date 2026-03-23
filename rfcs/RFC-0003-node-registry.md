> **Changelog — 2026-03-23:** DFOS integration — Nodes now boot as DFOS relays. Node registration is creating an identity chain on the relay, not a custom registration protocol. There is no separate MJN node registration handshake; joining the DFOS relay network is the join operation. See new section: [DFOS Integration: Nodes as Relays](#dfos-integration-nodes-as-relays).

---

# RFC-0003: Node Type Registry and Certification

| Field | Value |
|-------|-------|
| RFC | 0003 |
| Title | Node Type Registry and Certification |
| Author | Ryan Veteze (b0b) \<ryan@imajin.ai\> |
| Status | DRAFT |
| Created | 2026-02-25 |
| Updated | 2026-03-23 |
| Depends on | RFC-0001, RFC-0002 |

---

## Abstract

This document defines the MJN Node Type Registry — a classification system for the four types of nodes that can operate on the MJN network. It specifies the capabilities, requirements, and certification criteria for each node type.

Each MJN node is a **JBOS instance** — a Just a Bunch Of Services deployment running on the cryptographic kernel defined in RFC-0001. The node type classification describes what userspace services and governance model that instance exposes, not the underlying kernel (auth + pay + attestation/settlement), which is uniform across all node types.

## Motivation

RFC-0001 defines four node types: `person`, `community`, `operator`, and `infrastructure`. This RFC formalizes what each type means, what capabilities they have, and how nodes prove their type.

The registry serves three purposes:
1. **Interoperability** — Nodes know what capabilities to expect from each type
2. **Trust signals** — Users can make informed decisions about which nodes to trust
3. **Network health** — The Foundation can monitor node distribution and identify concentration risks

## Specification

### Node Type Definitions

#### Type: `person`

**Definition:** A node controlled by a single natural human.

**Requirements:**
- MUST have exactly one controlling DID
- MUST support personal identity attestations
- MUST support `.fair` manifest creation and signing
- SHOULD support profile publication
- SHOULD support trust graph connections

**Use cases:**
- Personal identity and presence
- Creative work publication
- Personal knowledge catalogue
- Direct peer-to-peer transactions

**TBD:** What prevents a person node from being operated by an AI agent? Should there be human verification requirements?

---

#### Type: `community`

**Definition:** A node controlled by multiple natural humans operating as a collective.

**Requirements:**
- MUST have multiple controlling DIDs
- MUST define governance rules (threshold signatures, consensus mechanisms, etc.)
- MUST support shared `.fair` manifest creation
- MUST declare member DIDs publicly or privately based on community preference

**Use cases:**
- Family nodes
- Event/conference nodes
- Research group nodes
- Cooperative organization nodes

**TBD:** Governance model requirements, decision-making transparency, member dispute resolution.

---

#### Type: `operator`

**Definition:** A service node that provides MJN-compatible services to other nodes.

**Requirements:**
- MUST implement full MJN protocol (all four primitives)
- MUST publish service terms and pricing
- MUST handle settlement atomically
- MUST provide uptime SLA declarations
- MUST support Federation (ability to interoperate with other operators)

**Use cases:**
- Platform integrations (Substack, 1Password, Anthropic)
- MJN-native applications
- Inference services
- Content hosting and distribution

**Examples:**
- imajin.ai (reference implementation)
- Hypothetical: Substack integration node
- Hypothetical: AI training data marketplace node

**TBD:** Certification requirements, compliance monitoring, penalty mechanisms for protocol violations.

---

#### Type: `infrastructure`

**Definition:** A protocol infrastructure node operated for network health, not commercial service.

**Requirements:**
- MUST be operated by the MJN Foundation or certified delegates
- MUST provide core protocol services (DID resolution, registry, relay)
- MUST boot as a DFOS relay (see DFOS Integration section)
- MUST NOT charge fees for basic protocol operations
- MUST publish operational transparency reports

**Use cases:**
- Foundation-operated resolver nodes
- Protocol relay nodes (DFOS relay + MJN discovery)
- Emergency failover infrastructure
- Network monitoring and health services

**TBD:** Delegation criteria, operational requirements, foundation oversight procedures.

---

### Node Certification

**Certification levels:**

1. **Self-declared** — Node operator declares type in DID Document, no third-party verification
2. **Community-verified** — Node type attested by trust graph connections
3. **Foundation-certified** — Node type verified by MJN Foundation (required for `infrastructure` type)

With DFOS integration, self-declared and community-verified certification is anchored to the DFOS identity chain of the node operator. A self-declared node type is a signed operation on the operator's chain; it is not just a claim in a DID Document. The chain record makes it auditable and tamper-evident.

**TBD:** Full certification procedures, verification criteria, appeal mechanisms.

### Node Registry

**Requirements:**
- The MJN Foundation MUST operate a public node registry
- Registry MUST be queryable by DID, node type, service capabilities
- Registry MUST NOT be required for basic protocol operation (decentralization requirement)
- Registry entries MUST include: DID, node type, certification level, service endpoints, last verified date

**TBD:** Registry data format, update procedures, retention policies.

## DFOS Integration: Nodes as Relays

### Node Boot as DFOS Relay

With DFOS integration, MJN nodes boot as DFOS relays. This changes how node registration works:

**Before DFOS integration (prior model):**
- Nodes registered through a custom MJN registration protocol
- Node presence was announced to the federated registry via a separate handshake
- TBD: Full registration protocol (was an open question)

**After DFOS integration (current model):**
- A node boots by standing up a DFOS relay
- There is no separate MJN node registration step — joining the DFOS relay network IS the join operation
- Node identity is the relay's own DFOS identity chain
- Node presence in the federated registry is the relay's presence in the DFOS relay network

### Registration as Identity Chain Creation

When a new identity registers on a DFOS-backed MJN node, the registration operation is:

1. The user generates (or imports) an Ed25519 keypair
2. The node relay creates a DFOS identity chain for that keypair — this is the genesis operation
3. The chain genesis block contains the identity scope declaration, initial key roles, and node anchor reference
4. The `did:imajin` alias is registered, pointing to the new chain

No custom MJN registration protocol is required. The signed chain genesis IS the registration. Any relay that can reach the DFOS network can verify it.

**What this eliminates:**
- Custom registration handshake (was TBD — no longer needed)
- Separate announcement mechanism (relay network handles this)
- Platform-specific account creation (keypair + chain genesis = presence)

### Node Type Declaration on Chain

The four node types (`person`, `community`, `operator`, `infrastructure`) are declared as signed operations on the node operator's DFOS identity chain:

```json
{
  "operation": "nodeType.declare",
  "chainId": "dfos:chain:node-operator-123",
  "nodeType": "operator",
  "certificationLevel": "self-declared",
  "capabilities": ["full-mjn-protocol", "inference", "settlement"],
  "serviceEndpoint": "https://node.example.com/mjn",
  "signature": "<ed25519-sig-by-controller-key>",
  "timestamp": "2026-03-23T00:00:00Z"
}
```

The chain record makes the declaration:
- **Auditable:** The full history of type declarations is on the chain
- **Tamper-evident:** Any modification breaks the chain
- **Portable:** Any node can verify the declaration against the operator's public key

### All Nodes vs Only Infrastructure

All node types benefit from DFOS relay architecture. However:

- **`infrastructure` nodes** MUST boot as DFOS relays
- **`operator` nodes** SHOULD boot as DFOS relays (enables full federation)
- **`community` and `person` nodes** MAY boot as DFOS relays or operate through an existing relay

A `person` node that is not running its own relay is still a valid MJN node — it anchors its identity chain to a trusted relay and operates through it.

---

## Rationale

### Why distinguish node types at all?

Without type classification, users have no signal about what to expect from a node. Is this DID a person I can message, or infrastructure I can query? Type classification provides minimum capability guarantees.

### Why not more granular types?

More types = more complexity. Four types cover the primary use cases while remaining simple enough for broad adoption.

### Why allow self-declaration?

Requiring third-party verification for all nodes creates gatekeeping and centralization. Self-declaration preserves permissionless participation. Trust graph attestation and Foundation certification provide verification for nodes that want higher trust signals.

## Security Considerations

### Node Type Spoofing

A malicious actor could declare themselves as a `person` node but operate as an automated service or AI agent.

**Mitigations:**
- Trust graph attestation (peers vouch for node type)
- Behavioral analysis (automated detection of bot-like patterns)
- Foundation certification for high-trust scenarios

**TBD:** Full spoofing detection mechanisms.

### Infrastructure Capture

If all `infrastructure` nodes are operated by a single entity, the network becomes centralized.

**Mitigation:** Foundation MUST delegate infrastructure operation to geographically and jurisdictionally diverse operators.

**TBD:** Delegation criteria, diversity requirements, monitoring procedures.

### Registry as Single Point of Failure

If the registry is required for protocol operation, it becomes a centralization and failure risk.

**Mitigation:** Registry MUST be informational only, not required for basic MJN operations. Nodes can operate without registry lookup.

## Open Questions

1. **AI agent nodes:** Should there be a node type for AI agents? Or should AI agents operate under `operator` type with disclosure requirements?

2. **Organization nodes:** Should there be a fifth type for corporations, foundations, institutions? Or do they fit under `operator` or `community`?

3. **Hybrid nodes:** Can a single DID operate as multiple node types? (E.g., a person who also runs an operator service)

4. **Type migration:** If a `person` node evolves into an `operator` node, what's the migration path? Can DIDs change types?

5. **Minimum viable certification:** What's the lightest possible certification that still provides meaningful trust signals?

6. **Registry governance:** Should the registry be operated exclusively by the Foundation, or should there be multiple competing registries?

## Related RFCs

| RFC | Title | Status |
|-----|-------|--------|
| RFC-0001 | MJN Core Specification | DRAFT |
| RFC-0002 | did:mjn Method Specification | DRAFT |
| RFC-0003 | Node Type Registry (this document) | DRAFT |
| RFC-0005 | Trust Graph and Sybil Resistance | DRAFT |

## Reference Implementation

**TBD:** Link to reference implementation when available.

Reference implementation will include:
- Node type declaration utilities
- Registry query API
- Certification verification tools
- Example node configurations for each type

## Copyright

CC0 1.0 Universal. No rights reserved.

---

**Community input requested on:**
- AI agent node type requirements
- Organization vs community distinction
- Type migration procedures
- Registry governance model

Discussion: [github.com/ima-jin/mjn-protocol/issues](https://github.com/ima-jin/mjn-protocol/issues)

---

*Ryan Veteze (b0b) · ryan@imajin.ai · 2026-03-23 (DFOS integration update)*
