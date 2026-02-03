# OpenARK

## Status of this Document

This document specifies **BOLT-ARK**, a proposed extension to the Lightning Network protocol suite. It is an experimental specification intended for research, interoperability testing, and discussion. It does not modify existing BOLT consensus rules and introduces no new Bitcoin consensus changes.

The keywords **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119.

---

## Abstract

OpenARK defines an extension to Lightning enabling **VTXO-based multi-party channels** with a designated **Ark Service Provider (ASP)** acting as a resolver and cosigner. The construction preserves HTLC semantics, supports RGB assets, and uses **Nostr** as the transport layer for off-chain coordination messages. The protocol allows many users to share a single on-chain root while retaining unilateral exit guarantees.

---

## 1. Introduction

ARK extends Lightning by allowing multiple participants to share a **single on-chain funding root**, from which a **VTXO tree** is derived. State transitions are cosigned by an Ark Service Provider while preserving unilateral exit guarantees for users.

---

## 2. Goals and Non-Goals

### Goals
- Preserve HTLC compatibility with Lightning
- Enable many-user shared liquidity via VTXO trees
- Support RGB asset state transitions
- Use Nostr as a censorship-resistant transport
- Maintain unilateral exit at all times

### Non-Goals
- Removing Bitcoin enforcement
- Custodial fund control by the ASP

---

## 3. Terminology

- **ASP (Ark Service Provider)**:  
  Entity coordinating rounds and cosigning transitions.

- **User**:  
  Participant owning one or more VTXO series.

- **Co-Verifier**:  
  Optional third party holding one of the ASP threshold signing keys.

- **VTXO (Virtual Transaction Output)**:  
  A **finite, ordered series of off-chain transactions** that begins at a **vtxo-leaf**, traverses zero or more
  **vtxo-branches**, reaches a **vtxo-trunk** as the first off-chain transaction, and is **anchored on-chain**
  by a **vtxo-root output**.

- **vtxo-leaf**:  
  The **final off-chain output** in a VTXO series. The vtxo-leaf represents the current spendable state and is
  the output that a user MAY unilaterally commit on-chain by broadcasting the required transaction path.

- **vtxo-branch**:  
  An **intermediate off-chain transaction** in a VTXO series that links a vtxo-leaf output to the vtxo-trunk
  and encodes a valid state transition.

- **vtxo-trunk**:  
  The **first off-chain transaction** in a VTXO series, aggregating one or more vtxo-branches and serving as
  the off-chain predecessor to the vtxo-root transaction.

- **vtxo-root transaction**:  
  An **on-chain Bitcoin transaction** that anchors one or more vtxo-trunks and establishes on-chain
  enforceability of all descendant VTXO series.

- **Forfeit Transaction**:  
  A presigned off-chain transaction that forms part of the **forfeit tree**. Forfeit transactions atomically
  bind ownership of a closed-round VTXO series to a successor round or recycling path. The forfeit tree consists of:
  1. A **forfeit control transaction** defining forfeiture conditions.
  2. One or more **forfeit leaf transactions** mapping individual VTXO series to their successor state.

- **Recycle Transaction**:  
  An on-chain Bitcoin transaction used to recover value for inactive or offline users after the recycle block
  height has been reached. Two forms exist:
  - **Cooperative recycle transaction**
  - **Unilateral recycle transaction**

- **Round**:  
  A bounded time window in which VTXO transitions occur.

A VTXO **MUST** be interpreted strictly as an ordered path  
**vtxo-leaf → vtxo-branch(es) → vtxo-trunk → vtxo-root**  
and **MUST NOT** be interpreted as a single output, balance, or UTXO-like object.

---

## 4. System Model

The system consists of:
- Bitcoin L1 for final settlement
- Lightning-compatible HTLC semantics
- A Nostr relay network for coordination
- An ASP providing synchronization but not custody

Trust assumptions:
- The ASP MAY censor protocol messages.
- The ASP MUST NOT unilaterally access, freeze, or steal user funds.
- Users MUST be able to exit unilaterally at any time.

---

## 5. Cryptographic Primitives

- Schnorr signatures (BIP-340)
- MuSig2 for aggregate signing
- Hashlocks and timelocks (BOLT-compatible)
- Threshold signatures for ASP + Co-Verifiers

---

## 6. VTXO Model

A VTXO is a finite, ordered series of off-chain transactions representing a claim to value under defined spending conditions.

A user MAY unilaterally exit by broadcasting the vtxo-leaf output together with the required ancestor transactions up to and including the on-chain vtxo-root transaction.

A VTXO series consists of:
- A terminal vtxo-leaf output.
- Zero or more vtxo-branches linking the leaf to the trunk.
- A vtxo-trunk transaction anchored by a vtxo-root transaction.

Together, all VTXO series form a directed acyclic graph (the **vtxo-tree**).

---

## 7. Round Lifecycle

Each ARK round progresses through a deterministic lifecycle bounded by Bitcoin block height.

### 7.1 Round States

Initiated → Started → Closed → Recycled

- **Initiated**: Participant discovery and tree construction.
- **Started**: Off-chain transitions permitted.
- **Closed**: No new off-chain transitions.
- **Recycled**: Recovery and termination.

### 7.1.1 Initiated State

Substates: Invite → Confirm → Fund

### 7.1.2 Started State

Entered when the vtxo-root is confirmed on-chain and `new_round_start` is broadcast.

### 7.1.3 Closed State

Entered at the closing block. All outstanding HTLC requests MUST be aborted.

### 7.1.4 Recycled State

Entered when a recycle transaction is broadcast, either cooperatively or unilaterally.

---

## 7.3 HTLC Transitions

HTLC transitions follow a request/accept/complete pattern analogous to Lightning.

HTLCs always lock the **vtxo-leaf output** of a VTXO series.

---

## 8. HTLC Semantics in ARK

(HTLC state diagrams unchanged)

---

## 9. Message Transport (Nostr)

Requirements:
- Messages MUST be signed by the sender
- Events MUST reference the round ID
- Relays MUST be treated as untrusted

---

## Two-tier Security – Cloud Agents

Cloud agents are optional components assisting users with monitoring and round participation.

They MAY:
- Monitor for unilateral exits
- Participate in round transitions

They MUST NOT:
- Control unilateral exit
- Hold sufficient key material to steal funds

---

## 10. Funding, Forfeit, and Recycle Transactions

ARK uses three transaction classes:
- Funding transactions
- Forfeit transactions
- Recycle transactions

Each class is defined normatively in Section 3.

---

## 11. On-chain Enforcement and Unilateral Exit

At any time, a user MAY enforce ownership by broadcasting the vtxo-leaf path on-chain.

All VTXO series MUST map to a valid Bitcoin spending path.

---

## 12. RGB Integration

ARK supports RGB by binding asset state transitions to VTXO transitions.

Asset and BTC transitions MUST be atomic.

---

## 13. Privacy Considerations

- VTXO ownership is off-chain
- ASP learns graph structure but not intent
- Nostr metadata leakage MUST be considered

---

## 14. Security Considerations

Primary threats:
- ASP censorship
- Relay censorship
- Key compromise

Mitigations:
- Unilateral exits
- Time-bounded rounds
- Threshold signing

---

## 15. Failure Modes and Recovery

- Offline users recover via recycle transactions
- ASP failure triggers unilateral exits
- Co-Verifiers reduce single-operator risk

---

## 16. Compatibility with Lightning (BOLT 2–11)

ARK preserves Lightning HTLC semantics and routing compatibility while introducing a higher-layer construction.

---

## 17. Deployment Considerations

- ASPs SHOULD publish reliability metrics
- Users SHOULD limit exposure per round
- Multiple ASPs MAY coexist

---

## 18. Acknowledgements

This design draws inspiration from Lightning, channel factories, and ARK research.

---

## 19. Examples

(Examples unchanged)
