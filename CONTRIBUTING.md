# Contributing to MJN

MJN is an open protocol. Anyone can propose a change, raise a problem, or submit an RFC. The protocol belongs to the community of now persons who implement and use it.

---

## How the RFC Process Works

RFCs (Request for Comments) are how MJN evolves. Every change to the protocol — new primitives, extensions, corrections, security fixes — goes through an RFC.

### Five Stages

```
PROPOSED → DRAFT → REVIEW → ACCEPTED → SUPERSEDED
```

**PROPOSED** — An Issue is opened describing the problem and a rough solution. Community discussion begins. No document required yet.

**DRAFT** — A document is written using the RFC template and submitted as a Pull Request to `/rfcs/`. The RFC number is assigned at this stage. The author iterates based on community feedback.

**REVIEW** — The Technical Council reviews the RFC for correctness, completeness, and compatibility with existing RFCs. A minimum 14-day review window applies to all RFCs. Security-critical RFCs require 30 days.

**ACCEPTED** — The RFC is merged into `/rfcs/`. It is now part of the protocol. Implementations are expected to conform.

**SUPERSEDED** — An older RFC has been replaced by a newer one. The original remains in the repository for reference, marked as superseded.

---

## RFC Template

Copy this template to `/rfcs/RFC-XXXX-short-name.md` when opening a Draft PR.

```markdown
# RFC-XXXX: Title

| Field | Value |
|-------|-------|
| RFC | XXXX |
| Title | |
| Author | Name \<email\> |
| Status | DRAFT |
| Created | YYYY-MM-DD |
| Supersedes | (if applicable) |
| Superseded by | (if applicable) |

## Abstract

One paragraph summary.

## Motivation

What problem does this solve? Why does it need to be at the protocol layer?

## Specification

The normative content. Use RFC 2119 keywords (MUST, SHOULD, MAY, etc.).

## Rationale

Why this approach over alternatives?

## Security Considerations

What are the attack surfaces? What assumptions does this make?

## Open Questions

What is unresolved? What does the community need to weigh in on?

## Reference Implementation

Link to implementation if it exists.

## Copyright

CC0 1.0 Universal. No rights reserved.
```

---

## What Makes a Good RFC

**Start with the problem, not the solution.** The best RFCs begin by precisely describing a failure mode in the current protocol. The solution follows from the problem.

**Be normative where it matters.** Use MUST for requirements that break interoperability if ignored. Use SHOULD for strong recommendations. Use MAY for optional extensions. Vague language produces vague implementations.

**Address the attack surface.** Every protocol primitive has adversarial cases. If your RFC doesn't discuss them, reviewers will ask.

**Resolve open questions before ACCEPTED.** A question left open in a DRAFT is an invitation for community input. A question left open in an ACCEPTED RFC is a bug.

---

## What Does Not Require an RFC

- Bug fixes in the reference implementation (imajin.ai) — file an issue there
- Documentation improvements — open a PR directly
- Tooling, SDKs, example implementations — no process required

---

## RFC Numbering

RFCs are numbered sequentially. Numbers are assigned when a DRAFT PR is opened. Numbers are never reused. Gaps in the sequence indicate withdrawn proposals — these are noted in the RFC index.

Current RFC index: [rfcs/README.md](rfcs/README.md)

---

## Code of Conduct

This is a community built on the premise that authentic human connection and sovereign presence matter. Discussions should reflect that.

Be direct. Be honest. Disagree on the merits. Don't be cruel.

The protocol is non-extractive by design. So is this community.

---

*今人 — the protocol belongs to the now persons who use it.*
