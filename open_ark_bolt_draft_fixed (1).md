# OpenARK

## Status of this Document

This document specifies **BOLT-ARK**, a proposed extension to the Lightning Network protocol suite. It is an experimental specification intended for research, interoperability testing, and discussion. It does not modify existing BOLT consensus rules and introduces no new Bitcoin consensus changes.

The keywords **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119.

---

## Abstract

OpenARK defines an extension to Lightning enabling **VTXO-based multi-party channels** with a designated **Ark Service Provider (ASP)** acting as a resolver and cosigner. The construction preserves HTLC semantics, supports RGB assets, and uses **Nostr** as the transport layer for off-chain coordination messages. The protocol allows many users to share a single on-chain root while retaining unilateral exit guarantees.

---

## Table of Contents

1. Introduction
2. Goals and Non-Goals
3. Terminology
4. System Model
5. Cryptographic Primitives
6. VTXO Model
7. Round Lifecycle
8. Message Transport (Nostr)
9. HTLC Semantics in ARK
10. Funding, Forfeit, and Recycle Transactions
11. On-chain Enforcement and Unilateral Exit
12. RGB Integration
13. Privacy Considerations
14. Security Considerations
15. Failure Modes and Recovery
16. Compatibility with Lightning (BOLT 2–11)
17. Reference Message Formats
18. Deployment Considerations
19. Acknowledgements

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

- **ASP (Ark Service Provider)**: Entity coordinating rounds and cosigning transitions
- **User**: Participant owning a VTXO leaf
- **Co-Verifier**: Optional third party holding one of the ASP threshold keys
- **VTXO**: Virtual Transaction Output, representing off-chain ownership
- **Round**: A bounded time window in which VTXO transitions occur
- **Root Transaction**: On-chain transaction anchoring a round

---

## 4. System Model

The system consists of:
- Bitcoin L1 for final settlement
- Lightning-compatible HTLC semantics
- A Nostr relay network for coordination
- An ASP providing availability but not custody

Trust assumptions:
- ASP MAY censor but cannot steal funds
- Users MUST be able to exit unilaterally

---

## 5. Cryptographic Primitives

- Schnorr signatures (BIP-340)
- MuSig2 for aggregate signing
- Hashlocks and timelocks (BOLT-compatible)
- Threshold signatures for ASP + Co-Verifiers

---

## 6. VTXO Model

A VTXO is a logical output representing claim to value under a spending condition, that can be unilaterally committed on-chain by transmitting a series of presigned transactions that are:

- Terminated by an output that is the last step in the spending path and is called the vtxo-leaf.
- Bound via a set of presigned transactions, called vtxo-branches that are keept off-chain to an on-chain transaction called the vtxo-root.
- Redeemable on-chain by broadcasting the transactions, this is called unilateral exit.

Together all the VTXOs form a directed acyclic graph (vtxo-tree).

---

## 7. Round Lifecycle

Each ARK round progresses through a well-defined set of states. Rounds are **time-bounded by Bitcoin block height** and advance deterministically based on protocol messages and on-chain conditions.

### 7.1 Round States

The lifecycle consists of the following states:

- **Initiated**
  The ASP announces a new round and accepts participant registrations.

- **Started**  
  The VTXO tree has been finalized, the round vtxo-root is anchored on-chain, and off-chain transitions may occur.

- **Closed**  
  The closing block has been found. No new off-chain state transitions are permitted. Participants may prepare for exit or transfer funds to a new round.

- **Recycled**  
  The round is terminated. Remaining value is recovered via a recycle transaction, allowing inactive or offline participants to reclaim funds.
  This can either happen when all users aswell as the ASP have signed and broadcast the recycle transaction (collaborative recycling) or when the recycling block has been found (unilateral recycling),

### 7.2 State Transitions

The nominal message flow is:

1. `new_round_initiate` — ASP announces a new round (Initiated)
2. `new_round_join` — Users register participation
3. `new_round_vtxo_tree_proposal` — ASP proposes the VTXO tree
4. `new_round_vtxo_tree_accept` — Users cosign the tree
5. `new_round_prepare_start` — Final preparation before activation
6. `new_round_start` — Round enters Started state
7. `round_closed` — Round enters Closed state
8. `round_recycled` — Round enters Recycled state

### 7.3 Block Height Bounds

Two block-height parameters define round progression:

- **Closing Block**  
  The Bitcoin block height at which the round MUST transition from *Started* to *Closed*. After this point, no new off-chain transitions are allowed.

- **Recycle Block**  
  The Bitcoin block height at which the ASP MAY unilaterally sign and broadcast the recycle transaction, transitioning the round to *Recycled*.

These bounds guarantee liveness while preserving unilateral exit guarantees for all users.

---

## 8. Message Transport (Nostr)

ARK messages are transported over Nostr events.

Requirements:
- Messages MUST be signed by the sender
- Events MUST reference the round ID
- Relays MUST be treated as untrusted

This document defers detailed encoding to a companion **NIP-ARK** specification.

---

## 9. HTLC Semantics in ARK

ARK preserves Lightning HTLC semantics:

- Hashlock: `H(P)`
- Timelock: relative or absolute
- Resolution paths: success or timeout

HTLCs MAY be resolved:
- Off-chain via ASP cosignature
- On-chain via unilateral exit

---

## 10. Funding, Forfeit, and Recycle Transactions

### Funding
Users MAY fund rounds by:
- Transferring value from a previous round
- Adding new on-chain inputs

### Forfeit Transactions
Forfeit transactions atomically bind the transfer of ownership from the User to the ASP in a closed-round vtxo to a new-round vtxo-root. Making the root deposit an atomic transaction, enacting the forfeiture.   

### Recycle Transactions
Recycle transactions recover value to offline or inactive participants, enabling early termination of a round.

---

## 11. On-chain Enforcement and Unilateral Exit

At any time, a user MAY:
- Broadcast a unilateral exit transactions
- Claim their VTXO value on-chain by converting the VTXO to a UTXO.

All VTXOs MUST map to a valid on-chain spending path.

---

## 12. RGB Integration

ARK supports RGB by associating asset state transitions with VTXO transitions.

Requirements:
- RGB state MUST follow Bitcoin ownership
- Asset and BTC transitions MUST be atomic

Examples in Section 2 illustrate cross-asset HTLC swaps.

---

## 13. Privacy Considerations

- VTXO ownership is off-chain
- ASP learns graph structure but not intent
- Nostr metadata leakage MUST be considered

---

## 14. Security Considerations

Threats:
- ASP censorship
- Relay censorship
- Key compromise

Mitigations:
- Unilateral exits
- Time-bounded rounds
- Threshold signing

---

## 15. Failure Modes and Recovery

- Offline users are handled via recycle paths
- ASP failure triggers unilateral exits
- Co-Verifiers reduce single-operator risk

---

## 16. Compatibility with Lightning (BOLT 2–11)

ARK:
- Preserves HTLC behavior
- Does not alter gossip or routing
- Operates as an L2/L3 construction

Existing Lightning nodes are not required to understand ARK internals.

---

## 17. Reference Message Formats

Messages include:
- Type
- Round ID
- Payload
- Signature

Canonical encoding is defined in NIP-ARK.

---

## 18. Deployment Considerations

- ASPs SHOULD publish reliability metrics
- Users SHOULD limit exposure per round
- Multiple ASPs MAY coexist

---

## 19. Acknowledgements

This design draws inspiration from Lightning, channel factories, and the ARK research lineage.

