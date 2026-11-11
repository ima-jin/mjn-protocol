# MJN — iMaJiN Network Protocol

**identity and authority for AI agents.**

MJN is the protocol specification behind [Imajin](https://imajin.ai), the trust layer for AI agents.
Imajin is in alpha. The RFCs describe the protocol, not a guarantee that every feature is implemented.

MJN carries identity, attribution, consent, and settlement at the protocol layer.

MJN runs as **JBOS — Just a Bunch Of Services** on a cryptographic substrate. Independent HTTP services sit above a kernel of auth, pay, attestations, and `.fair` settlement — all backed by DFOS proof chains. Services are commodity. The chains are permanent.

---

## The Architecture: Scopes × Primitives

The protocol is organized around two dimensions. Every problem MJN solves — every interaction, every settlement — is a cell in the matrix formed by their intersection.

### Four Identity Scopes

| Scope | What It Is |
|-------|-----------|
| **Actor** | One DID, one keypair. The atomic unit. Humans, agents, and devices. |
| **Family** | Intimate trust. Shared resources, delegated authority. |
| **Community** | Shared purpose. Trust earned and attested. |
| **Business** | Structured entity. Roles, hierarchy, delegation chains. |

### Five Primitives

| Primitive | What It Carries |
|-----------|----------------|
| **Attestation** | Credentials, reputation, endorsement — cryptographically signed from the moment of issuance |
| **Communication** | Scoped messaging within and across trust rings |
| **Attribution** | .fair manifests, revenue chains, creative lineage — all manifests cryptographically signed |
| **Settlement** | Payments, fees, declared-intent marketplace |
| **Discovery** | Federated registry, node presence, queryable expertise |

The protocol IS the matrix. Consent is embedded throughout — in Attestation and Attribution — because consent without attribution is unenforceable, and attribution without consent is theft.

---

## Read the Spec

→ [RFC-0001: MJN Core Specification](rfcs/RFC-0001-mjn-core.md)
→ [RFC-0002: did:mjn Method Specification](rfcs/RFC-0002-did-mjn-method.md)

---

## Reference Implementation

The reference implementation is maintained by [imajin.ai](https://imajin.ai).

**14 live services**, self-hosted on owned hardware:

- **Core platform:** auth · profile · registry · connections · pay · events · chat · input · media · www
- **Applications:** coffee · dykil · links · learn
- Code: [github.com/ima-jin/imajin-ai](https://github.com/ima-jin/imajin-ai)
- First demonstration: April 1st, 2026

37 days. 73 registered identities. Real events with real ticket sales. One person with architectural clarity and the right tooling.

The reference implementation is one node among many. It does not define the protocol. The RFCs define the protocol.

---

## Governance

MJN is governed by the MJN Foundation, a Swiss *Stiftung* currently in formation. The Foundation holds the protocol specification, operates the RFC process, and governs token issuance under Swiss FINMA guidelines.

No entity — including imajin.ai and the MJN Foundation — has unilateral authority to modify an accepted RFC.

---

## Contribute

→ [How to propose an RFC](CONTRIBUTING.md)

Discussion happens in [Issues](https://github.com/ima-jin/mjn-protocol/issues). Everyone is welcome. The protocol belongs to the now persons who use it.

---

## License

CC0 1.0 Universal. No rights reserved. Public domain.

Any implementation of this specification is a valid MJN node. No license required. No permission needed.

---

*今人 — the sovereign human presence, here, now.*
