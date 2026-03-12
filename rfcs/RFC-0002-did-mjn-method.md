# RFC-0002: did:mjn Method Specification

| Field | Value |
|-------|-------|
| RFC | 0002 |
| Title | did:mjn Method Specification |
| Author | Ryan Veteze (b0b) \<ryan@imajin.ai\> |
| Status | DRAFT |
| Created | 2026-02-25 |
| Updated | 2026-03-12 |
| Depends on | RFC-0001 |

---

## Abstract

This document specifies the `did:mjn` method — the DID method used by all MJN identities for sovereign presence on the network. The method defines DID format, resolution, key types, DID Document structure, typed identity scopes, key rotation procedures, and recovery mechanisms.

## Motivation

RFC-0001 requires that every MJN identity be identified by a W3C-compliant DID using the `did:mjn` method. This RFC defines that method formally.

The `did:mjn` method MUST be:
- Cryptographically self-sovereign (no third-party issuance or revocation)
- Portable across all MJN implementations
- Resolvable without requiring a centralized registry
- Compatible with standard DID resolution libraries
- Typed — carrying its identity scope as a first-class property

## Specification

### DID Format

```
did:mjn:<multibase-encoded-public-key>
```

**TBD:** Full format specification, character set restrictions, validation rules.

### Typed DIDs — Identity Scope

Every `did:mjn` DID carries its identity scope in the DID Document. The scope is one of:

- **HumanActor** — A person. Self-sovereign. The irreducible unit.
- **AgentActor** — An AI agent. Child key derived from a parent DID. Authority scope encoded in key derivation path.
- **DeviceActor** — A physical device. Same cryptographic structure, same accountability chain.
- **Family** — Intimate trust group. Formed by mutual attestation between Actor DIDs.
- **Community** — Shared purpose group. Formed by quorum of founding Actor DIDs.
- **Business** — Structured entity. Formed by declaration from founding Actor DIDs.

The scope type MUST be declared in the DID Document `type` field. Implementations MUST verify that protocol operations are consistent with the declared scope — a Business DID cannot perform operations reserved for Actor DIDs, and vice versa.

### Actor Subtypes and Key Derivation

Actor DIDs cover three subtypes: HumanActor, AgentActor, and DeviceActor.

**AgentActor key derivation:** Agent DIDs are derived from a parent human or organizational DID using a deterministic key derivation path. The derivation path encodes the agent's authority scope — an agent key derived with scope `[attribution:collaborator]` cannot produce a valid signature claiming `[attribution:creator]`. Authority is structural, not runtime policy.

**DeviceActor key derivation:** Device DIDs follow the same derivation pattern as AgentActors — child keys derived from a parent DID with scoped authority.

**TBD:** Key derivation algorithm specification, scope encoding format, derivation path structure.

### DID Document Structure — Actor

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

### DID Document Structure — AgentActor

```json
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:mjn:z6AgentKey...",
  "type": "AgentActor",
  "controller": "did:mjn:z6ParentHumanDID...",
  "authorityScope": ["attribution:collaborator", "settlement:execute"],
  "verificationMethod": [{
    "id": "did:mjn:z6AgentKey...#keys-1",
    "type": "Ed25519VerificationKey2020",
    "controller": "did:mjn:z6ParentHumanDID...",
    "publicKeyMultibase": "z6AgentKey..."
  }],
  "authentication": ["did:mjn:z6AgentKey...#keys-1"],
  "service": [{
    "id": "did:mjn:z6AgentKey...#mjn-node",
    "type": "MJNNode",
    "serviceEndpoint": "https://node.example.com/mjn"
  }]
}
```

### DID Document Structure — Group Scopes (Family, Community, Business)

**TBD:** Full DID Document structure for group scope types. Key differences from Actor DIDs:
- Multi-key verification methods (quorum signing for Community, multi-sig for Family, delegation for Business)
- Founding anchor references (non-severable for Business)
- Membership roster references
- Governance model declaration

### Supported Key Types

The following key types are supported for `did:mjn` identifiers:

- **Ed25519** (REQUIRED) — Primary signing key type
- **secp256k1** (OPTIONAL) — For blockchain interoperability
- **X25519** (OPTIONAL) — For encryption

**Note on Solana compatibility:** Every Ed25519 keypair generated for MJN is already a valid Solana wallet. Every backup file is already a wallet private key. No integration required. The protocol wasn't designed to do this — it was excavated from building from the right principles.

**TBD:** Key encoding format, multicodec prefixes, key derivation procedures.

### DID Resolution

**TBD:**
- Resolution algorithm
- Scope type verification during resolution
- Caching requirements
- Resolution metadata
- Fallback behavior when resolution fails

### Key Rotation

**Requirements:**
- Key rotation MUST NOT invalidate historical signatures
- Old keys MUST remain verifiable for attribution chain integrity
- New keys MUST be announced via signed rotation document
- For group scopes, key rotation MUST respect the scope's governance model (consensus for Family, quorum for Community, delegation for Business)

**TBD:** Full key rotation procedure, rotation document format, announcement mechanism.

### Recovery Mechanisms

**Open question:** How should DID recovery work if a user loses their private key? Options:
1. Social recovery (threshold signatures from trusted contacts)
2. Designated recovery keys held separately
3. No recovery (key loss = identity loss, maximum sovereignty)

Community input needed on which approach best balances sovereignty and usability.

## Rationale

### Why not use did:key?

`did:key` is stateless and resolution-free, which is attractive. However, it doesn't support key rotation, service endpoints, or typed identity scopes. MJN identities need to advertise service endpoints, rotate keys without breaking attribution chains, and carry their scope type.

### Why not use did:web?

`did:web` ties identity to DNS infrastructure controlled by ICANN and nation-states. This violates the sovereignty requirement.

### Why not use did:ion or did:ethr?

Blockchain-based methods introduce dependency on specific chain infrastructure and transaction costs. MJN identity must be free to create and free to resolve.

### Why typed DIDs?

Every identity system in production today — Google, Apple, Meta, even W3C DID Core — models identity as one type and handles differences in application logic. MJN encodes the type at the protocol level because trust relationships, governance models, and privacy boundaries are structural. They cannot be safely handled by application code because application code can be bypassed. When the type is in the identity itself, the structural guarantees travel with it.

## Security Considerations

### Key Compromise

If a DID private key is compromised:
- Historical signatures remain valid (immutability requirement)
- Future signatures can be prevented via key rotation
- **Open question:** Should there be an emergency revocation mechanism?

### Agent Authority Escalation

AgentActor DIDs derive authority from key derivation scope. Implementations MUST verify that agent operations fall within the declared `authorityScope`. An agent attempting operations outside its scope MUST be rejected at the protocol level, not at the application level.

### DID Hijacking

**TBD:** Protection mechanisms against DID hijacking attacks.

### Resolution Attacks

**TBD:** Protection against malicious resolution responses, caching attacks, timing attacks.

### Group DID Capture

For Community DIDs: no single Actor DID can hold unilateral control — this is a structural requirement. For Business DIDs: the founding anchor is non-severable. Implementations MUST enforce these constraints at the DID layer.

## Open Questions

1. **Key rotation announcement:** How do nodes discover that a DID has rotated its keys? Push model vs pull model?

2. **Recovery vs sovereignty tradeoff:** Is social recovery compatible with cryptographic self-sovereignty? If a threshold of others can recover your identity, is it truly sovereign?

3. **Offline DID creation:** Can DIDs be created and used entirely offline, or do they require initial registration with at least one resolution node?

4. **Light client support:** What is the minimum viable DID Document for resource-constrained devices (DeviceActor on IoT/embedded systems)?

5. **Cross-chain interoperability:** Should `did:mjn` support optional blockchain anchoring for nodes that want additional immutability guarantees?

6. **Scope type migration:** Can an identity change scope? Can a soft Business DID (created by community check-ins) transition to a claimed Business DID? What are the rules?

7. **Agent derivation depth:** How many levels of agent derivation are permitted? Can an AgentActor derive a sub-agent?

## Related RFCs

| RFC | Title | Status |
|-----|-------|--------|
| RFC-0001 | MJN Core Specification | DRAFT |
| RFC-0002 | did:mjn Method Specification (this document) | DRAFT |
| RFC-0003 | Node Registry and Scope Certification | PLANNED |
| RFC-0005 | Trust Graph and Sybil Resistance | PLANNED |

## Reference Implementation

**TBD:** Link to reference implementation when available.

Reference implementation will include:
- DID creation utilities (all Actor subtypes)
- DID resolution library
- Scope type verification
- Key rotation tooling
- Key derivation for AgentActor and DeviceActor
- Example DID Documents for each identity scope

## Copyright

CC0 1.0 Universal. No rights reserved.

---

**Community input requested on:**
- Key rotation mechanisms
- Recovery approaches (social vs none vs designated keys)
- Offline DID creation requirements
- Light client specifications (especially DeviceActor)
- Agent key derivation scope encoding
- Group DID governance model encoding

Discussion: [github.com/ima-jin/mjn-protocol/issues](https://github.com/ima-jin/mjn-protocol/issues)

---

*Ryan Veteze (b0b) · ryan@imajin.ai · 2026-03-12*
