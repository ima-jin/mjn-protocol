# RFC-0007: Government Identity Attestation Layer

| Field | Value |
|-------|-------|
| RFC | 0007 |
| Title | Government Identity Attestation Layer |
| Author | Ryan Veteze (b0b) \<ryan@imajin.ai\> |
| Status | DRAFT |
| Created | 2026-02-25 |
| Depends on | RFC-0001, RFC-0002, RFC-0005 |

---

## Abstract

This document specifies how government-issued digital identity credentials interact with the MJN protocol's self-sovereign DID architecture. It defines a **layered identity model** where government DIDs attest to sovereign MJN DIDs without becoming the root of trust.

Government-issued identity adds standing and regulatory compliance to MJN nodes while preserving cryptographic self-sovereignty. Revocation of government credentials does not destroy the underlying sovereign identity — the same way losing your passport doesn't make you not a person.

## Motivation

Government-issued digital identity is not hypothetical. It's already being deployed:

- **EU eIDAS 2.0** — Digital identity wallet, mandatory for member states by 2026
- **Estonia e-Residency** — Government-issued digital identity for non-residents
- **India Aadhaar** — Biometric identity system for 1.3 billion people
- **Canada Digital ID** — Provincial digital identity frameworks (BC, Ontario, Alberta)
- **US mDL (mobile Driver's License)** — State-issued digital credentials

Every government implementation shares the same architectural characteristic: **revocable by design**. The government is the issuer and the revoker. This creates:

- **Centralized trust root** — Government becomes single point of failure
- **Political capture risk** — Governments can revoke identity for dissent, non-payment, political reasons
- **Cross-border fragmentation** — Your Canadian government DID isn't recognized in the US
- **Platform lock-in** — Government chooses the infrastructure, users have no alternative

MJN does not reject government identity. MJN **gives government identity somewhere sovereign to live**.

The solution is layered identity:

```
Government DID (revocable, state-issued)
    ↓ attests to
MJN DID (sovereign, cryptographically yours)
    ↓ operates on
MJN network
```

Your government DID becomes **one attestation among many** on your sovereign MJN node. It adds standing — "this person is a verified Canadian citizen" — without being the root. If the government revokes it, your node still exists. You just lose that particular attestation.

## Specification

### Layered Identity Architecture

#### Layer 1: Sovereign Identity (did:mjn)

Your **self-sovereign DID** is the root. You control the private key. No entity can issue it or revoke it. This is your persistent identity across all contexts, jurisdictions, and platforms.

```
did:mjn:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK
```

**Properties:**
- Cryptographically self-sovereign
- Portable across all implementations
- Not revocable by third parties
- Permanent (exists as long as you hold the key)

---

#### Layer 2: Government Attestation

A **government-issued DID** (or credential) attests to your sovereign DID. The government doesn't issue your identity — it vouches for claims about your identity.

**Attestation format:**

```json
{
  "version": "1.0",
  "attester": "did:gov:ca",
  "subject": "did:mjn:z6Mk...",
  "attestationType": "government-identity",
  "claims": {
    "citizenshipCountry": "CA",
    "legalName": "Jane Doe",
    "dateOfBirth": "1990-01-15",
    "governmentID": "CAN-123456789",
    "issuingAuthority": "Service Canada",
    "validFrom": "2026-01-01",
    "validUntil": "2036-01-01",
    "revocationEndpoint": "https://identity.canada.ca/revocation"
  },
  "timestamp": "2026-02-25T00:00:00Z",
  "signature": "<government-key-signature>"
}
```

**Key properties:**
- The government **attests to** your DID, it does not **issue** your DID
- The attestation is **time-bounded** (valid from/until dates)
- The attestation is **revocable** (via government-controlled revocation endpoint)
- Revocation of attestation does NOT revoke your `did:mjn`

---

#### Layer 3: Trust Signals

Your sovereign DID now carries **government attestation** as one trust signal among many:

- Government attestation: "This is a verified Canadian citizen"
- Employer attestation: "This person works for Acme Corp"
- Community attestation: "This person is a trusted member of the Toronto dev community"
- Platform attestation: "This person has a verified GitHub account with 10 years of history"

No single attestation is the root of trust. The **combination** of attestations creates a trust profile. If government revokes their attestation (visa expires, citizenship changes, etc.), the other attestations remain.

---

### Government DID Format

Government DIDs use the `did:gov` method (or country-specific methods like `did:ca`, `did:eu`, `did:in`).

**Format:**
```
did:gov:<country-code>:<unique-identifier>
```

**Examples:**
```
did:gov:ca                          (Government of Canada root DID)
did:gov:ca:service-canada           (Service Canada issuing authority)
did:ca:passport:CAN-123456789       (Specific passport credential)
```

**TBD:** Should governments use `did:gov` or country-specific methods (`did:ca`, `did:eu`)? What's the namespace governance?

---

### Government Attestation Use Cases

#### Use Case 1: KYC/AML Compliance

A financial service operator node requires government identity verification for regulatory compliance.

**Flow:**
1. User connects with sovereign `did:mjn`
2. Operator queries: "Do you have a government attestation for citizenship?"
3. User presents government attestation (or declines)
4. Operator verifies attestation signature against government's public key
5. Operator checks revocation endpoint (is this still valid?)
6. If valid, operator proceeds with KYC-compliant service

**Critical:** The operator **never sees the government DID private key**. They only see the attestation signed by the government vouching for the sovereign DID.

---

#### Use Case 2: Cross-Border Recognition

Alice has a Canadian government attestation. She travels to the EU and wants to use MJN-enabled services.

**Flow:**
1. Alice's `did:mjn` carries Canadian government attestation
2. EU service recognizes `did:gov:ca` as a trusted issuer (via mutual recognition treaty)
3. EU service accepts attestation as proof of identity
4. Alice operates with her sovereign DID, government attestation provides regulatory compliance

**Alternative:** If the EU and Canada don't have mutual recognition, Alice can get a **separate EU government attestation** on the same sovereign DID. Now she has two government attestations, usable in different jurisdictions, all anchored to one sovereign identity.

---

#### Use Case 3: Revocation Doesn't Break Identity

Bob's government ID expires (passport renewal delayed). His government attestation is revoked.

**What breaks:**
- Services requiring active government ID (banking, border crossing)

**What doesn't break:**
- Bob's sovereign `did:mjn` (still exists)
- Bob's other attestations (employer, community, platforms)
- Bob's `.fair` creative catalogue (still owned, still earns)
- Bob's trust graph connections (still valid)

Bob renews his passport → gets new government attestation → links it to same `did:mjn` → services requiring government ID work again.

**Key insight:** Temporary loss of government standing doesn't destroy digital existence. This is the difference between sovereign identity and platform identity.

---

### Privacy Considerations

**Problem:** Government attestations can enable surveillance. If every MJN transaction reveals government ID, governments can track all activity.

**Solution: Selective Disclosure**

Government attestations support **zero-knowledge proofs** and **selective disclosure**. You can prove claims without revealing the full attestation.

**Examples:**

**Prove age without revealing birthdate:**
```
User proves: "I am over 18"
Without revealing: Exact date of birth, government ID number
```

**Prove citizenship without revealing identity:**
```
User proves: "I am a Canadian citizen"
Without revealing: Legal name, ID number, address
```

**Prove residency without revealing location:**
```
User proves: "I live in the EU"
Without revealing: Specific country, city, address
```

**TBD:** Full zero-knowledge proof mechanisms for government attestations, selective disclosure protocols.

---

### Revocation Handling

Government attestations are revocable. The attestation includes a `revocationEndpoint` — a URL where the revocation status can be checked.

**Revocation check:**
```
GET https://identity.canada.ca/revocation/CAN-123456789
Response:
{
  "revoked": false,
  "validUntil": "2036-01-01"
}
```

**Revocation reasons:**
- Credential expired (passport, visa, work permit)
- Credential invalidated (passport reported stolen)
- Citizenship/residency status changed
- Government error or fraud detected

**Node behavior on revocation:**
- The government attestation is marked as invalid
- The sovereign `did:mjn` remains valid
- Services requiring government attestation will fail
- Services not requiring government attestation continue to work

**Open question:** Should revoked government attestations be hidden from the trust graph, or remain visible with "revoked" status for transparency?

---

### Multi-Jurisdiction Identity

A single sovereign DID can carry attestations from multiple governments.

**Example:**
```json
{
  "did": "did:mjn:z6Mk...",
  "attestations": [
    {
      "attester": "did:gov:ca",
      "claims": { "citizenshipCountry": "CA", ... },
      "status": "valid"
    },
    {
      "attester": "did:gov:eu",
      "claims": { "residency": "DE", ... },
      "status": "valid"
    },
    {
      "attester": "did:gov:in",
      "claims": { "workPermit": "Bangalore Tech Zone", ... },
      "status": "expired"
    }
  ]
}
```

**Use cases:**
- Dual citizenship
- International residents (citizen of one country, resident of another)
- Digital nomads (multiple residency permits)
- Refugees (recognized by host country, not home country)

---

## Rationale

### Why not make government DID the root identity?

If government DID is the root, then:
- Government can revoke your entire digital existence
- Cross-border operation requires government permission
- Political dissidents lose their identity when governments revoke it
- Platform lock-in (whichever platform the government chose)

MJN inverts this: **your identity is yours, government vouches for it**.

### Why allow government attestation at all?

Because governments provide real value:
- Regulatory compliance (KYC/AML requirements are real)
- Physical-digital bridge (governments verify you exist in meatspace)
- Legal accountability (courts recognize government ID)
- Border crossing (passports aren't going away)

Rejecting government identity entirely makes MJN unusable for regulated industries. Layering government attestation on sovereign identity gets the best of both: compliance where needed, sovereignty everywhere else.

### Why not use Verifiable Credentials (W3C VC) for government attestations?

We should! W3C Verifiable Credentials are the right standard for government attestations. This RFC is compatible with VCs — government attestations can be issued as VCs and presented to services via standard VC protocols.

**TBD:** Full integration with W3C VC spec, presentation protocols, revocation registries.

---

## Security Considerations

### Government Key Compromise

If a government's signing key is compromised, attackers can issue fraudulent attestations.

**Mitigation:**
- Governments MUST use hardware security modules (HSMs) for signing keys
- Governments SHOULD rotate keys regularly
- Old attestations signed by compromised keys MUST be revoked
- Emergency revocation endpoint for mass invalidation

**TBD:** Key rotation procedures, emergency revocation protocols.

---

### Coerced Disclosure

Governments may compel users to disclose their sovereign DID private keys.

**This is out of scope for the protocol.** MJN cannot prevent physical coercion any more than HTTPS can prevent rubber-hose cryptanalysis. But:

- Sovereign DID + government attestation means losing government ID doesn't lose your identity
- Users can have multiple sovereign DIDs for different contexts (one for government-facing services, one for private activity)
- Zero-knowledge proofs minimize what governments can compel users to reveal

---

### Surveillance via Attestation Metadata

Government attestations may include tracking metadata (serial numbers, issuance timestamps, location data).

**Mitigation:**
- Users SHOULD be able to selectively disclose claims without revealing full attestation
- Zero-knowledge proofs allow proving "I have a valid Canadian passport" without revealing passport number
- Services SHOULD only request the minimum claims necessary (e.g., "Are you over 18?" not "What's your birthdate?")

**TBD:** Privacy-preserving attestation presentation protocols.

---

### Government Censorship via Revocation

Authoritarian governments may revoke attestations for political reasons (dissidents, journalists, protesters).

**This is a feature, not a bug.**

If governments can revoke attestations arbitrarily, users **still have their sovereign DID**. They lose access to government-required services (banking, border crossing) but they don't lose:
- Their creative catalogue (`.fair` manifests)
- Their trust graph connections
- Their ability to operate on non-regulated parts of the network
- Their ability to get attestations from other governments (asylum/refugee status)

The layered model ensures government revocation is **limited in blast radius**. Losing government standing hurts, but it's not existential.

---

## Open Questions

1. **DID method namespace:** Should governments use `did:gov:ca` or `did:ca`? Who governs the namespace?

2. **Mutual recognition:** How do governments establish trust in each other's attestations? Is there a global registry of trusted government issuers?

3. **Stateless persons:** What attestation do refugees, stateless persons, or those fleeing persecution get? Is there a "recognized by UN" attestation type?

4. **Attestation expiry grace periods:** If a passport expires but renewal is in progress, should there be a grace period where the attestation remains valid?

5. **Government node types:** Should governments operate as `infrastructure` nodes, `operator` nodes, or a new `government` node type?

6. **Revocation permanence:** If a government revokes an attestation in error, can it be un-revoked? Or is revocation permanent and requires re-issuance?

7. **Cross-chain government attestations:** If governments issue attestations on different blockchains (EU on Ethereum, Canada on Solana), how do they interoperate?

8. **Privacy vs compliance tradeoff:** How do we balance user privacy (minimal disclosure) with government requirements (full KYC data)?

9. **Biometric binding:** Should government attestations optionally bind to biometric data for high-security contexts? How is that data stored/protected?

10. **Corporate identity:** Should corporations, NGOs, and legal entities get government attestations on their MJN DIDs? (e.g., "This DID is registered to Acme Corp, Delaware C-corp")

---

## Related RFCs

| RFC | Title | Status |
|-----|-------|--------|
| RFC-0001 | MJN Core Specification | DRAFT |
| RFC-0002 | did:mjn Method Specification | DRAFT |
| RFC-0003 | Node Type Registry and Certification | DRAFT |
| RFC-0005 | Trust Graph and Sybil Resistance | DRAFT |
| RFC-0007 | Government Identity Attestation Layer (this document) | DRAFT |

---

## Reference Implementation

**TBD:** Link to reference implementation when available.

Reference implementation will include:
- Government attestation issuance toolkit
- Attestation verification library
- Revocation status checking
- Zero-knowledge proof generation for selective disclosure
- Example integrations with eIDAS 2.0, mDL standards

---

## Appendix A: Real-World Government Identity Systems

| System | Country | Status | DID-compatible? |
|--------|---------|--------|-----------------|
| eIDAS 2.0 | EU | Mandatory 2026 | Potentially (uses W3C VCs) |
| e-Residency | Estonia | Active since 2014 | No (proprietary) |
| Aadhaar | India | Active, 1.3B users | No (centralized DB) |
| mDL (mobile Driver's License) | USA (state-level) | Pilot programs | Potentially (ISO 18013-5 standard) |
| Digital ID | Canada (provincial) | Pilots in BC, ON, AB | No (proprietary) |
| MyInfo | Singapore | Active since 2016 | No (centralized) |

**Opportunity:** MJN provides the interoperability layer these systems currently lack. A Canadian Digital ID attestation and an EU eIDAS attestation can both vouch for the same sovereign `did:mjn`.

---

## Appendix B: The Passport Analogy

Think of government attestation like a passport:

- **Losing your passport** doesn't make you not a person — it just makes one proof of personhood temporarily unavailable
- **Your passport expires** every 10 years, but you don't stop existing — you renew it
- **Different countries issue different passports** to the same person (dual citizenship) — each is valid in different contexts
- **A passport gets you across borders** but it's not your identity — it's proof of identity issued by a trusted authority

Government attestation on MJN works the same way:

- **Losing government attestation** (revoked, expired) doesn't destroy your `did:mjn` — it just makes one attestation unavailable
- **Government attestations expire** and get renewed — your sovereign DID persists across renewals
- **Multiple governments can attest** to the same sovereign DID — each attestation is valid in different jurisdictions
- **Government attestation enables regulated services** (banking, border crossing) but it's not your root identity — it's an attestation on your root identity

The difference: With a passport, if you lose it abroad, you're stuck until you can prove who you are to get a replacement. With MJN, if you lose government attestation, **you still have your sovereign DID and all your other attestations**. You can operate in non-regulated contexts while you sort out the government credential.

---

## Copyright

CC0 1.0 Universal. No rights reserved.

---

**Community input CRITICAL on:**
- DID method namespace for governments (did:gov vs did:ca)
- Mutual recognition frameworks for cross-border attestations
- Privacy-preserving disclosure mechanisms
- Stateless persons and refugee attestation models
- Government node type classification

Discussion: [github.com/ima-jin/mjn-protocol/issues](https://github.com/ima-jin/mjn-protocol/issues)

---

*Ryan Veteze (b0b) · ryan@imajin.ai · 2026-02-25*
