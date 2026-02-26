# RFC-0006: .fair Manifest Extensions

| Field | Value |
|-------|-------|
| RFC | 0006 |
| Title | .fair Manifest Extensions |
| Author | Ryan Veteze (b0b) \<ryan@imajin.ai\> |
| Status | DRAFT |
| Created | 2026-02-25 |
| Depends on | RFC-0001 |

---

## Abstract

This document defines extension mechanisms for the `.fair` attribution manifest beyond the core schema specified in RFC-0001. Extensions allow creators to add domain-specific metadata, custom usage terms, and experimental fields without breaking core protocol compatibility.

## Motivation

RFC-0001 defines a minimal `.fair` manifest schema for identity, contribution shares, usage terms, and settlement. But creative work is diverse:

- A music track may need **sampling credits** and **performance royalties**
- A software library may need **dependency attribution** and **security vulnerability disclosures**
- A scientific paper may need **dataset citations** and **reproducibility metadata**
- A physical design may need **fabrication instructions** and **material sourcing**

`.fair` must be extensible to support domain-specific needs while maintaining a simple, universal core that all MJN nodes understand.

## Specification

### Extension Format

Extensions are added to the `.fair` manifest in an `extensions` object:

```json
{
  "version": "1.0",
  "did": "did:mjn:...",
  "contributors": [...],
  "terms": {...},
  "extensions": {
    "extensionName": {
      "version": "1.0",
      "data": {}
    }
  },
  "signature": "..."
}
```

**Requirements:**
- Extensions MUST be optional (nodes that don't understand an extension can safely ignore it)
- Extensions MUST NOT alter core .fair semantics (contributor shares, settlement distribution)
- Extensions MUST be versioned independently from core .fair spec
- Extensions SHOULD be defined in separate RFC documents

---

### Proposed Extensions

#### Extension: Music Sampling

For music tracks that sample other works.

```json
"extensions": {
  "music-sampling": {
    "version": "1.0",
    "samples": [
      {
        "sourceDID": "did:mjn:...",
        "sourceTitle": "Original Track Name",
        "sampleDuration": 3.5,
        "sampleTimestamp": "0:45-0:48.5",
        "clearanceStatus": "licensed | fair-use | pending"
      }
    ],
    "performanceRights": {
      "mechanicalRoyalties": true,
      "publicPerformance": true,
      "streamingRoyalties": true
    }
  }
}
```

**TBD:** Full sampling attribution schema, clearance workflows, royalty split calculations.

---

#### Extension: Software Dependencies

For code libraries and software projects.

```json
"extensions": {
  "software-dependencies": {
    "version": "1.0",
    "dependencies": [
      {
        "name": "library-name",
        "version": "1.2.3",
        "did": "did:mjn:...",
        "license": "MIT",
        "criticalPath": true
      }
    ],
    "securityAudits": [
      {
        "auditor": "did:mjn:...",
        "date": "2026-01-15",
        "reportURL": "https://...",
        "vulnerabilitiesFound": 0
      }
    ]
  }
}
```

**TBD:** Dependency resolution protocols, vulnerability disclosure workflows, license compatibility verification.

---

#### Extension: Academic Citations

For research papers and scientific publications.

```json
"extensions": {
  "academic-citations": {
    "version": "1.0",
    "citations": [
      {
        "sourceDID": "did:mjn:...",
        "citationType": "methodology | dataset | theory | review",
        "relevance": 0.0-1.0
      }
    ],
    "datasets": [
      {
        "did": "did:mjn:...",
        "format": "CSV | JSON | HDF5",
        "size": "1.2 GB",
        "reproducibilityHash": "sha256:..."
      }
    ],
    "peerReview": {
      "reviewers": ["did:mjn:...", "did:mjn:..."],
      "journal": "Journal Name",
      "accepted": "2026-02-01"
    }
  }
}
```

**TBD:** Citation graph construction, peer review workflows, dataset reproducibility verification.

---

#### Extension: Physical Fabrication

For hardware designs, physical art, manufactured goods.

```json
"extensions": {
  "physical-fabrication": {
    "version": "1.0",
    "materials": [
      {
        "name": "Birch plywood",
        "source": "Sustainable Forestry Certified",
        "quantity": "2 sheets, 4'x8'x0.75\"",
        "cost": 85.00,
        "currency": "USD"
      }
    ],
    "fabricationSteps": [
      {
        "step": 1,
        "instruction": "CNC router: cut outer frame using template-outer.dxf",
        "tool": "CNC router, 1/4\" endmill",
        "duration": "45 minutes"
      }
    ],
    "safetyWarnings": [
      "Wear eye protection during routing",
      "Dust mask required for sanding"
    ]
  }
}
```

**TBD:** BOM (bill of materials) schema, fabrication instruction formats, safety metadata.

---

#### Extension: AI Training Provenance

For content used to train AI models — track *what models were trained on this*, not just *this work was created by AI*.

```json
"extensions": {
  "ai-training-provenance": {
    "version": "1.0",
    "trainedModels": [
      {
        "modelDID": "did:mjn:...",
        "modelName": "Claude 5.0",
        "trainingDate": "2027-03-15",
        "consentDID": "did:mjn:...",
        "settlementTxHash": "0x..."
      }
    ],
    "trainingTerms": {
      "exclusivity": false,
      "modelVersionRestrictions": "none",
      "geographicRestrictions": "none"
    }
  }
}
```

**TBD:** Model training tracking, consent audit trails, derivative model attribution chains.

---

### Extension Lifecycle

**PROPOSED** → **DRAFT** → **ACCEPTED** → **DEPRECATED**

1. **PROPOSED:** Community proposes extension via GitHub Issue
2. **DRAFT:** Extension schema is written and published (no RFC number required for extensions)
3. **ACCEPTED:** Extension is stable, implementations exist, community adopts
4. **DEPRECATED:** Extension is superseded by better approach, remains for backward compatibility

Extensions do NOT require RFC numbers unless they modify core protocol behavior. They can evolve independently.

---

### Extension Namespacing

To prevent naming conflicts, extensions SHOULD use reverse-domain naming:

```json
"extensions": {
  "com.imajin.hardware-attribution": {...},
  "org.soundcloud.streaming-royalties": {...},
  "edu.mit.research-data": {...}
}
```

**Open question:** Should there be a centralized registry of extension names, or is community coordination sufficient?

---

### Extension Signing

Extensions are covered by the manifest's root signature. Tampering with extension data breaks the signature, same as tampering with core fields.

**Open question:** Should individual extensions be separately signable by domain experts? (E.g., a music sampling extension co-signed by the original artist and the sampling artist)

---

## Rationale

### Why extensions instead of multiple manifest formats?

Multiple manifest formats fragment the ecosystem. A music `.fair` manifest would be incompatible with a software `.fair` manifest. Extensions keep the core universal while allowing domain-specific enrichment.

### Why allow optional extensions?

Requiring all nodes to understand all extensions creates implementation burden and slows protocol evolution. Optional extensions allow experimentation without breaking existing implementations.

### Why not use JSON-LD or RDF?

JSON-LD provides rich semantic extensibility but at the cost of complexity. The `.fair` manifest is designed for simplicity and broad adoption. Extensions provide 80% of the value of JSON-LD at 20% of the complexity.

---

## Security Considerations

### Malicious Extensions

A malicious extension could include:
- Tracking pixels or privacy-violating metadata
- Malformed data that crashes parsers
- Misleading claims (fake security audits, fraudulent certifications)

**Mitigation:**
- Nodes SHOULD validate extension schemas before processing
- Nodes SHOULD sandbox extension parsing (don't trust extension data)
- Community reputation for extension authors

**TBD:** Extension validation tooling, sandboxing requirements.

---

### Extension Bloat

Unrestricted extensions could make `.fair` manifests arbitrarily large, creating storage and bandwidth costs.

**Mitigation:**
- **Recommended max manifest size:** 100 KB for embedded manifests
- Large extension data (e.g., full fabrication instructions) SHOULD link to external resources via DID references, not embed inline

**TBD:** Size limits, external reference requirements.

---

## Open Questions

1. **Extension registry:** Should there be a centralized registry of known extensions, or is decentralized coordination enough?

2. **Extension versioning:** What happens when an extension schema changes? How do nodes handle version mismatches?

3. **Multi-signature extensions:** Should extensions support separate signatures from domain experts (e.g., music sampling cleared by both artist and sampler)?

4. **Extension conflicts:** What if two extensions have conflicting semantics? (E.g., one says "no commercial use", another says "royalty-free commercial")

5. **Backward compatibility:** If an extension is deprecated, do old manifests with that extension remain valid?

6. **Extension discovery:** How do users/nodes discover what extensions are available and what they mean?

7. **Mandatory extensions:** Should certain extensions be mandatory for certain content types? (E.g., physical goods MUST have fabrication metadata?)

---

## Related RFCs

| RFC | Title | Status |
|-----|-------|--------|
| RFC-0001 | MJN Core Specification | DRAFT |
| RFC-0006 | .fair Manifest Extensions (this document) | DRAFT |

---

## Reference Implementation

**TBD:** Link to reference implementation when available.

Reference implementation will include:
- Extension schema validator
- Example extensions for common domains (music, code, research, hardware)
- Extension authoring toolkit
- Extension registry (if community decides to build one)

---

## Appendix A: Example Extended Manifest

Full `.fair` manifest for a hardware design (8×8×8 LED cube) with fabrication extension:

```json
{
  "version": "1.0",
  "did": "did:mjn:z6Mk...",
  "created": "2026-02-25T00:00:00Z",
  "contributors": [
    { "did": "did:mjn:z6Mk...", "role": "author", "share": 0.90 },
    { "did": "did:mjn:z6Ab...", "role": "fabricator", "share": 0.10 }
  ],
  "derivedFrom": [],
  "terms": {
    "read":         { "price": 0.00, "currency": "free" },
    "train":        { "price": "prohibited" },
    "redistribute": { "price": 0.00, "conditions": "attribution-required" },
    "remix":        { "price": 0.00, "conditions": "attribution-required" }
  },
  "canonical": "https://imajin.ai/designs/unit-8x8x8",
  "extensions": {
    "physical-fabrication": {
      "version": "1.0",
      "materials": [
        {
          "name": "LED strips (WS2812B)",
          "quantity": "64 strips × 8 LEDs",
          "source": "Adafruit",
          "cost": 280.00,
          "currency": "USD"
        },
        {
          "name": "Birch plywood frame",
          "quantity": "2 sheets 4'×8'×0.75\"",
          "source": "Home Depot",
          "cost": 85.00,
          "currency": "USD"
        }
      ],
      "fabricationSteps": [
        {
          "step": 1,
          "instruction": "CNC router: cut frame using frame-template.dxf",
          "tool": "CNC router, 1/4\" endmill",
          "duration": "45 minutes"
        },
        {
          "step": 2,
          "instruction": "Solder LED strips to controller board",
          "tool": "Soldering iron, lead-free solder",
          "duration": "3 hours"
        }
      ],
      "safetyWarnings": [
        "Wear eye protection during CNC operation",
        "Ventilation required for soldering"
      ],
      "files": [
        { "name": "frame-template.dxf", "did": "did:mjn:z6Cd..." },
        { "name": "pcb-gerber.zip", "did": "did:mjn:z6Ef..." }
      ]
    }
  },
  "signature": "..."
}
```

---

## Copyright

CC0 1.0 Universal. No rights reserved.

---

**Community input requested on:**
- Extension registry vs decentralized coordination
- Multi-signature extension support
- Extension conflict resolution
- Domain-specific extension proposals (gaming, fashion, food, etc.)

Discussion: [github.com/ima-jin/mjn-protocol/issues](https://github.com/ima-jin/mjn-protocol/issues)

---

*Ryan Veteze (b0b) · ryan@imajin.ai · 2026-02-25*
