# OpenARK

## Status of this Document

This document specifies **BOLT-ARK**, a proposed extension to the Lightning Network protocol suite.

This document is **normative**, except where explicitly marked as *informative*.
It is an experimental specification intended for research, interoperability testing, and discussion.
It does not modify existing BOLT consensus rules and introduces no new Bitcoin consensus changes.

The keywords **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119.

---

## Abstract

OpenARK defines an extension to Lightning enabling **VTXO-based multi-party channels** with a designated
**Ark Service Provider (ASP)** acting as a resolver and cosigner.

The protocol **MUST preserve HTLC semantics**, **MUST support unilateral exit**, and **MAY support RGB assets**.
Off-chain coordination messages **MAY** be transported using **Nostr**.

---

## 1. Introduction

ARK extends Lightning by allowing multiple participants to share a **single on-chain funding root**,
from which **VTXO series** are derived.

State transitions **MUST** be cosigned by an Ark Service Provider and **MUST NOT** compromise
users’ unilateral exit guarantees.

---

## 2. Goals and Non-Goals (Informative)

### Goals (Informative)

The following goals describe non-normative design objectives:

- Preserve HTLC compatibility with Lightning
- Enable many-user shared liquidity via VTXO trees
- Support RGB asset state transitions
- Use censorship-resistant transports
- Maintain unilateral exit guarantees

### Non-Goals (Informative)

- Removing Bitcoin enforcement
- Custodial fund control by the ASP

---

## 3. Terminology (Normative)

- **ASP (Ark Service Provider)**:  
  An entity coordinating rounds and cosigning transitions.

- **User**:  
  A participant owning one or more VTXO series.

- **Co-Verifier**:  
  An optional third party holding one of the ASP threshold signing keys.

- **VTXO (Virtual Transaction Output)**:  
  A **finite, ordered series of off-chain transactions** that begins at a **vtxo-leaf**, traverses one or more
  **vtxo-branches**, reaches a **vtxo-trunk** as the first off-chain transaction, and is **anchored on-chain**
  by a **vtxo-root transaction**.

- **vtxo-leaf**:  
  The **final off-chain transaction** in a VTXO series, representing the current spendable state and the
  transaction that a user MAY unilaterally commit on-chain.

- **vtxo-branch**:  
  An **intermediate off-chain transaction** in a VTXO series that links a vtxo-leaf to the vtxo-trunk and
  encodes a valid state transition.

- **vtxo-trunk**:  
  The **first off-chain transaction** in a VTXO series, aggregating one or more vtxo-branches and serving as
  the off-chain predecessor to the vtxo-root transaction.

- **vtxo-root transaction**:  
  An **on-chain Bitcoin transaction** that anchors one or more vtxo-trunks and establishes on-chain
  enforceability of all descendant VTXO series.

- **Round**:  
  A bounded time window in which VTXO transitions occur.

A VTXO **MUST** be interpreted strictly as an ordered path
**vtxo-leaf → vtxo-branch(es) → vtxo-trunk → vtxo-root**
and **MUST NOT** be interpreted as a single output, balance, or UTXO-like object.

---

## 4. System Model (Normative)

The system consists of:

- Bitcoin Layer 1 for final settlement
- Lightning-compatible HTLC semantics
- An untrusted message relay network
- An ASP providing availability but not custody

### Trust Assumptions

- The ASP **MAY** censor protocol messages
- The ASP **MUST NOT** be able to steal user funds
- Users **MUST** be able to execute a unilateral exit at any time

---

## 5. Cryptographic Primitives (Normative)

- Schnorr signatures (BIP-340)
- MuSig2 for aggregate signing
- Hashlocks and timelocks compatible with BOLT semantics
- Threshold signatures for ASP + Co-Verifiers

---

## 6. VTXO Model (Normative)

All VTXO behavior **MUST** conform to the definitions in Section 3 (Terminology).

A user’s current spendable state **MUST** be represented by the **vtxo-leaf** of a VTXO series.

Unilateral exit **MUST** be performed by broadcasting the vtxo-leaf together with all ancestor
transactions up to and including the vtxo-root transaction.

All VTXO series **MUST** collectively form a directed acyclic graph (the **vtxo-tree**).

---

## 7. Round Lifecycle (Normative)

Rounds **MUST** be time-bounded by Bitcoin block height and **MUST** advance deterministically.

### 7.1 Round States

A round **MUST** transition through the following states:

- **Initiated**
- **Started**
- **Closed**
- **Recycled**

Invalid transitions **MUST NOT** occur.

---

## 7.3 HTLC Transitions (Normative)

HTLCs **MUST** lock and transition the **vtxo-leaf of a VTXO series**.

HTLC resolution **MUST** preserve Lightning safety and liveness properties.

---

## 9. Message Transport (Nostr) (Normative)

- Messages **MUST** be signed by the sender
- Events **MUST** reference the round identifier
- Relays **MUST** be treated as untrusted

Transport encoding is specified in a companion **NIP-150** document.

---

## Two-Tier Security – Cloud Agents (Informative)

This section is **informative and non-normative**.

Users **MAY** deploy always-on agents to assist with monitoring and round transitions.
Such agents **MUST NOT** be required for protocol correctness.

---

## 10. Funding, Forfeit, and Recycle Transactions (Normative)

Users **MAY** fund rounds via previous rounds or new on-chain inputs.

Forfeit and recycle transactions **MUST** preserve reclaimability for all users under all failure modes.

---

## 11. On-chain Enforcement and Unilateral Exit (Normative)

At any time, a user **MAY** initiate unilateral exit by committing a vtxo-leaf on-chain.

All VTXO series **MUST** map to a valid on-chain spending path.

---

## 19. Examples (Informative)

This section is **non-normative** and provided for illustration only.
