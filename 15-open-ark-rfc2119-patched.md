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
from which a **VTXO tree** is derived.

State transitions **MUST** be cosigned by an Ark Service Provider while **MUST NOT** compromising
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

- **ASP (Ark Service Provider)**: An entity coordinating rounds and cosigning transitions
- **User**: A participant owning a VTXO leaf
- **Co-Verifier**: An optional third party holding one of the ASP threshold keys
- **VTXO**: A Virtual Transaction Output representing off-chain ownership
- **Round**: A bounded time window in which VTXO transitions occur
- **Root Transaction**: An on-chain transaction anchoring a round

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

A **VTXO** **MUST** represent a claim to value under a defined spending condition.

A VTXO **MUST** be bound to an on-chain root via a finite sequence of presigned transactions
(the **vtxo-branches**) terminating in a **vtxo-leaf**.

A VTXO **MUST** be redeemable on-chain by broadcasting the associated transactions,
providing unilateral exit.

All VTXOs **MUST** form a directed acyclic graph (the **vtxo-tree**).

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

#### 7.1.1 Initiated State

The Initiated state **MUST** include the substates: Invite, Confirm, and Fund.

##### Invite Substate

The ASP **MUST** announce a new round using `new_round_initiate`.

Users **MAY** respond with `new_round_join` messages.

The ASP **MUST** wait until either `min_timeout` or `max_timeout` is reached before transitioning
to the Confirm substate.

##### Confirm Substate

The ASP **MUST** broadcast `new_round_vtxo_tree_proposal`.

Users **MUST** verify the proposal and **MAY** respond with `new_round_vtxo_tree_accept`.

Once all required signatures are collected, the protocol **MUST** transition to the Fund substate.

##### Fund Substate

The ASP **MUST** broadcast `new_round_prepare_start`.

Users **MUST** provide signatures for all required transactions using `new_round_start_prepared`.

Once complete, the ASP **MUST** sign and broadcast the vtxo-root transaction.

##### Aborting Initialization

A user **MAY** abort onboarding at any time by issuing `new_round_abort`.

Aborted funds **MUST** be handled via the recycle path or carried forward.

---

## 7.3 HTLC Transitions (Normative)

OpenARK **MUST** preserve Lightning HTLC safety and liveness properties.

HTLC transitions **MUST** follow a request / accept / complete pattern using
`vtxo_spend_request`, `vtxo_spend_accept`, and `vtxo_spend_complete`.

Abort conditions **MUST** be signaled using `vtxo_spend_failed`.

---

## 9. Message Transport (Nostr) (Normative)

- Messages **MUST** be signed by the sender
- Events **MUST** reference the round ID
- Relays **MUST** be treated as untrusted

Transport encoding details are specified in a companion **NIP-150** document.

---

## Two-Tier Security – Cloud Agents (Informative)

This section is **informative and non-normative**.

Users **MAY** deploy always-on agents to assist with monitoring and round transitions.
Such agents **MUST NOT** be required for protocol correctness.

---

## 10. Funding, Forfeit, and Recycle Transactions (Normative)

Users **MAY** fund rounds via previous rounds or new on-chain inputs.

Forfeit and recycle transactions **MUST** preserve user reclaimability under all failure modes.

---

## 11. On-chain Enforcement and Unilateral Exit (Normative)

At any time, a user **MAY** broadcast a unilateral exit transaction.

All VTXOs **MUST** map to a valid on-chain spending path.

---

## 19. Examples (Informative)

This section is **non-normative** and provided for illustration only.

