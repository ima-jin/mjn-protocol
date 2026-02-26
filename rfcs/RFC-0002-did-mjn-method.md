# RFC-0002: did:mjn Method Specification

| Field | Value |
|-------|-------|
| RFC | 0002 |
| Title | did:mjn Method Specification |
| Author | Ryan Veteze (b0b) \<ryan@imajin.ai\> |
| Status | DRAFT |
| Created | 2026-02-25 |
| Depends on | RFC-0001 |

---

## Abstract

This document specifies the `did:mjn` method — the DID method used by all MJN nodes for sovereign identity. The method defines DID format, resolution, key types, DID Document structure, key rotation procedures, and recovery mechanisms.

## Motivation

RFC-0001 requires that every MJN node be identified by a W3C-compliant DID using the `did:mjn` method. This RFC defines that method formally.

The `did:mjn` method MUST be:
- Cryptographically self-sovereign (no third-party issuance or revocation)
- Portable across all MJN implementations
- Resolvable without requiring a centralized registry
- Compatible with standard DID resolution libraries

## Specification

### DID Format

```
did:mjn:<multibase-encoded-public-key>
```

**TBD:** Full format specification, character set restrictions, validation rules.

### Supported Key Types

The following key types are supported for `did:mjn` identifiers:

- **Ed25519** (REQUIRED) — Primary signing key type
- **secp256k1** (OPTIONAL) — For blockchain interoperability
- **X25519** (OPTIONAL) — For encryption

**TBD:** Key encoding format, multicodec prefixes, key derivation procedures.

### DID Document Structure

**TBD:**
- Minimal DID Document requirements
- Service endpoint specifications
- Verification method requirements
- Authentication vs assertion methods

### DID Resolution

**TBD:**
- Resolution algorithm
- Caching requirements
- Resolution metadata
- Fallback behavior when resolution fails

### Key Rotation

**Requirements:**
- Key rotation MUST NOT invalidate historical signatures
- Old keys MUST remain verifiable for attribution chain integrity
- New keys MUST be announced via signed rotation document

**TBD:** Full key rotation procedure, rotation document format, announcement mechanism.

### Recovery Mechanisms

**Open question:** How should DID recovery work if a user loses their private key? Options:
1. Social recovery (threshold signatures from trusted contacts)
2. Designated recovery keys held separately
3. No recovery (key loss = identity loss, maximum sovereignty)

Community input needed on which approach best balances sovereignty and usability.

## Rationale

### Why not use did:key?

`did:key` is stateless and resolution-free, which is attractive. However, it doesn't support key rotation or service endpoints. MJN nodes need to advertise service endpoints and rotate keys without breaking attribution chains.

### Why not use did:web?

`did:web` ties identity to DNS infrastructure controlled by ICANN and nation-states. This violates the sovereignty requirement.

### Why not use did:ion or did:ethr?

Blockchain-based methods introduce dependency on specific chain infrastructure and transaction costs. MJN identity must be free to create and free to resolve.

## Security Considerations

### Key Compromise

If a DID private key is compromised:
- Historical signatures remain valid (immutability requirement)
- Future signatures can be prevented via key rotation
- **Open question:** Should there be an emergency revocation mechanism?

### DID Hijacking

**TBD:** Protection mechanisms against DID hijacking attacks.

### Resolution Attacks

**TBD:** Protection against malicious resolution responses, caching attacks, timing attacks.

## Open Questions

1. **Key rotation announcement:** How do nodes discover that a DID has rotated its keys? Push model vs pull model?

2. **Recovery vs sovereignty tradeoff:** Is social recovery compatible with cryptographic self-sovereignty? If a threshold of others can recover your identity, is it truly sovereign?

3. **Offline DID creation:** Can DIDs be created and used entirely offline, or do they require initial registration with at least one resolution node?

4. **Light client support:** What is the minimum viable DID Document for resource-constrained devices (IoT, embedded systems)?

5. **Cross-chain interoperability:** Should `did:mjn` support optional blockchain anchoring for nodes that want additional immutability guarantees?

## Related RFCs

| RFC | Title | Status |
|-----|-------|--------|
| RFC-0001 | MJN Core Specification | DRAFT |
| RFC-0002 | did:mjn Method Specification (this document) | DRAFT |
| RFC-0003 | Node Type Registry and Certification | DRAFT |
| RFC-0005 | Trust Graph and Sybil Resistance | DRAFT |

## Reference Implementation

**TBD:** Link to reference implementation when available.

Reference implementation will include:
- DID creation utilities
- DID resolution library
- Key rotation tooling
- Example DID Documents for each node type

## Copyright

CC0 1.0 Universal. No rights reserved.

---

**Community input requested on:**
- Key rotation mechanisms
- Recovery approaches (social vs none vs designated keys)
- Offline DID creation requirements
- Light client specifications

Discussion: [github.com/ima-jin/mjn-protocol/issues](https://github.com/ima-jin/mjn-protocol/issues)

---

*Ryan Veteze (b0b) · ryan@imajin.ai · 2026-02-25*
