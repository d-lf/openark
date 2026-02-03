# Proposed Improvements for OpenARK Specification

This document captures ten concrete, actionable improvements identified during review of the OpenARK specification.  
The focus is on **specification rigor, implementability, and protocol clarity**, in line with BOLT-style standards.

---

## 1. Normative Language Consistency (RFC 2119 Hygiene)

- Audit the entire document for correct use of **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY**.
- Replace informal language (e.g. *should*, *can*) where normative behavior is intended.
- Explicitly label sections as **normative** or **informative** where appropriate.

---

## 2. Explicit Threat Model Section

- Add a dedicated *Threat Model* subsection early in the document.
- Enumerate adversaries (ASP, relays, co-verifiers, user agents).
- Define adversarial capabilities (censorship, delay, equivocation, key compromise).
- Clearly state non-goals and excluded threat classes.

---

## 3. Formal Definition of VTXO Validity

- Define when a VTXO is considered **valid**, **spent**, or **invalidated**.
- Specify exclusivity rules between competing VTXOs.
- State invariants relating the VTXO DAG to on-chain enforceability.

---

## 4. Explicit ASP Authority Boundaries

- Clearly separate ASP **liveness power** from **safety power**.
- Enumerate actions the ASP can perform versus those it cannot.
- Specify which failures lead to delay only and which trigger unilateral exit.

---

## 5. Machine-Checkable Round State Transitions

- Introduce a formal round state transition table.
- Define valid and invalid state transitions.
- Optionally include a state diagram for the full round lifecycle.

---

## 6. Protocol Message Registry

- Add a dedicated section enumerating all protocol messages.
- For each message, define:
  - Required fields
  - Required signatures
  - Preconditions
  - Resulting state transitions
- Keep message semantics separate from transport encoding.

---

## 7. Formal HTLC Correctness Properties

- Explicitly state safety and liveness guarantees for HTLCs.
- Define conditions for timeout, success, and abort paths.
- Clearly map these properties to Lightning HTLC semantics.

---

## 8. Explicit On-Chain Transaction Templates

- Describe funding, forfeit, and recycle transactions structurally.
- Specify inputs, outputs, timelocks, and signature requirements.
- Pseudo-templates are sufficient, mirroring BOLT #3 style.

---

## 9. Terminology Precision and Consistency

- Eliminate ambiguous or overloaded terminology.
- Enforce one canonical term per concept.
- Add clarification notes where confusion is likely.

---

## 10. Strengthened Examples with Explicit Outcomes

- Augment each example with:
  - Final balances
  - Preserved invariants
  - Available unilateral exits
  - Outcome under immediate ASP failure
- Treat examples as executable mental test vectors.

---

## Summary

These improvements aim to elevate the OpenARK document from a strong conceptual proposal to a **BOLT-grade protocol specification**, suitable for independent implementation and formal review.
