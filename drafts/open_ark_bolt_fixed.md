
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
8. HTLC Semantics in ARK  
9. Message Transport (Nostr)  
10. Funding, Forfeit, and Recycle Transactions  
11. On-chain Enforcement and Unilateral Exit  
12. RGB Integration  
13. Privacy Considerations  
14. Security Considerations  
15. Failure Modes and Recovery  
16. Compatibility with Lightning (BOLT 2–11)  
17. Deployment Considerations  
18. Acknowledgements  

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
- The ASP MAY censor but cannot steal funds  
- Users MUST be able to exit unilaterally  

---

## 5. Cryptographic Primitives

- Schnorr signatures (BIP-340)  
- MuSig2 for aggregate signing  
- Hashlocks and timelocks (BOLT-compatible)  
- Threshold signatures for ASP + Co-Verifiers  

---

## 6. VTXO Model

A VTXO is a logical output representing a claim to value under a spending condition that can be unilaterally committed on-chain by broadcasting a series of presigned transactions:

- Terminated by an output that is the last step in the spending path, called the **vtxo-leaf**.  
- Bound via a set of presigned transactions, called **vtxo-branches**, that connect off-chain state to an on-chain transaction called the **vtxo-root**.  
- Redeemable on-chain by broadcasting the transactions; this is called **unilateral exit**.  

Together, all VTXOs form a directed acyclic graph (the **vtxo-tree**).

---

## 7. Round Lifecycle

Each ARK round progresses through a well-defined set of states. Rounds are **time-bounded by Bitcoin block height** and advance deterministically based on protocol messages and on-chain conditions.

### 7.1 Round States

#### Initiated
The ASP announces a new round and accepts participant registrations.

#### Started
The VTXO tree has been finalized, the round vtxo-root is anchored on-chain, and off-chain transitions may occur.

#### Closed
The closing block has been reached. No new off-chain state transitions are permitted.

#### Recycled
The round is terminated and remaining value is recovered via the recycle transaction.

---

## 8. HTLC Semantics in ARK

ARK preserves Lightning HTLC semantics.

HTLCs MAY be resolved:
- Off-chain via ASP cosignature  
- On-chain via unilateral exit  

### HTLC State Machine

```mermaid
stateDiagram-v2
    [*] --> HTLC_Offer
    HTLC_Offer --> HTLC_Success: preimage revealed
    HTLC_Offer --> HTLC_Timeout: timelock expired
    HTLC_Success --> [*]
    HTLC_Timeout --> [*]
```

#### HTLC_Offer
Locks a VTXO under hashlock and timelock conditions and is signed by the sender and ASP.

#### HTLC_Success
Transfers the VTXO to the receiver upon presentation of the valid preimage. Signed by receiver, preimage, and ASP.

#### HTLC_Timeout
Returns the VTXO to the sender after timeout expiry. Signed by sender and ASP, optionally by the receiver.

---

## 9. Message Transport (Nostr)

ARK messages are transported over Nostr events.

Requirements:
- Messages MUST be signed by the sender  
- Events MUST reference the round ID  
- Relays MUST be treated as untrusted  

Detailed encoding is deferred to a companion **NIP-150** specification.

---

## 10. Funding, Forfeit, and Recycle Transactions

### Funding
Users MAY fund rounds by:
- Transferring value from a previous round  
- Adding new on-chain inputs  

### Forfeit Transactions
Forfeit transactions atomically bind ownership transfer from a closed-round VTXO to a new-round vtxo-root.

### Recycle Transactions
Recycle transactions recover value for offline or inactive participants and guarantee round termination.

---

## 11. On-chain Enforcement and Unilateral Exit

At any time, a user MAY:
- Broadcast a unilateral exit transaction  
- Claim VTXO value on-chain by converting it to a UTXO  

All VTXOs MUST map to a valid on-chain spending path.

---

## 12. RGB Integration

ARK supports RGB by associating asset state transitions with VTXO transitions.

Requirements:
- RGB state MUST follow Bitcoin ownership  
- Asset and BTC transitions MUST be atomic  

---

## 13. Privacy Considerations

- VTXO ownership is off-chain  
- The ASP learns graph structure but not intent  
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

## 17. Deployment Considerations

- ASPs SHOULD publish reliability metrics  
- Users SHOULD limit exposure per round  
- Multiple ASPs MAY coexist, interconnected via Lightning  

---

## 18. Acknowledgements

This design draws inspiration from Lightning, channel factories, and the ARK research lineage.

## 19. Example

![ARK Example](../OpenARK-example.svg)

### Roles   

- **Users**: Alice, Bob, Carol, Dave, Eve.
- **Ark Service Provider (ASP)**: Steve.
- **eXternal Liquidity Provider (XLP)**: Victoria.

### Example 1.0 - Alice, Bob, Carol, Dave and Eve create a VTXO tree

In this part of the example the users join a new ARK run by Steve, where Victoria acts as an XLP.

```mermaid
sequenceDiagram
    autonumber
    participant Steve as Steve (ASP)
    participant Victoria as Victoria (XLP)
    participant Alice as Alice (User)
    participant Bob as Bob (User)
    participant Carol as Carol (User)
    participant Dave as Dave (User)
    participant Eve as Eve (User)

    Note over Steve,Victoria: Round initiation (Victoria participates as XLP in this round)
    Steve->>Alice: new_round_initiate (round_id, params)
    Steve->>Bob: new_round_initiate (round_id, params)
    Steve->>Carol: new_round_initiate (round_id, params)
    Steve->>Dave: new_round_initiate (round_id, params)
    Steve->>Eve: new_round_initiate (round_id, params)
    Steve->>Victoria: new_round_initiate (round_id, params)

    Note over Alice,Eve: Users/XLP declare onboarding/offboarding/transition intents
    Alice->>Steve: new_round_join (onboard/offboard/transition)
    Bob->>Steve: new_round_join (onboard/offboard/transition)
    Carol->>Steve: new_round_join (onboard/offboard/transition)
    Dave->>Steve: new_round_join (onboard/offboard/transition)
    Eve->>Steve: new_round_join (onboard/offboard/transition)
    Victoria->>Steve: new_round_join (liquidity contribution)

    Note over Steve,Victoria: Steve proposes VTXO tree + related tx templates for signing
    Steve->>Alice: new_round_vtxo_tree_proposal (vtxo-tree, forfeit-tree, recycle txs)
    Steve->>Bob: new_round_vtxo_tree_proposal (vtxo-tree, forfeit-tree, recycle txs)
    Steve->>Carol: new_round_vtxo_tree_proposal (vtxo-tree, forfeit-tree, recycle txs)
    Steve->>Dave: new_round_vtxo_tree_proposal (vtxo-tree, forfeit-tree, recycle txs)
    Steve->>Eve: new_round_vtxo_tree_proposal (vtxo-tree, forfeit-tree, recycle txs)
    Steve->>Victoria: new_round_vtxo_tree_proposal (vtxo-tree, forfeit-tree, recycle txs)

    Note over Alice,Victoria: Participants verify and return signatures on required nodes
    Alice->>Steve: new_round_vtxo_tree_accept (signatures)
    Bob->>Steve: new_round_vtxo_tree_accept (signatures)
    Carol->>Steve: new_round_vtxo_tree_accept (signatures)
    Dave->>Steve: new_round_vtxo_tree_accept (signatures)
    Eve->>Steve: new_round_vtxo_tree_accept (signatures)
    Victoria->>Steve: new_round_vtxo_tree_accept (signatures)

    Note over Steve,Victoria: Steve binds tree to the new root and requests final prep signatures
    Steve->>Alice: new_round_prepare_start (root-to-sign, forfeit/recycle/onboarding to-sign)
    Steve->>Bob: new_round_prepare_start (root-to-sign, forfeit/recycle/onboarding to-sign)
    Steve->>Carol: new_round_prepare_start (root-to-sign, forfeit/recycle/onboarding to-sign)
    Steve->>Dave: new_round_prepare_start (root-to-sign, forfeit/recycle/onboarding to-sign)
    Steve->>Eve: new_round_prepare_start (root-to-sign, forfeit/recycle/onboarding to-sign)
    Steve->>Victoria: new_round_prepare_start (root-to-sign, forfeit/recycle/liquidity to-sign)

    Alice->>Steve: new_round_start_prepared (signatures: forfeit/recycle/onboarding)
    Bob->>Steve: new_round_start_prepared (signatures: forfeit/recycle/onboarding)
    Carol->>Steve: new_round_start_prepared (signatures: forfeit/recycle/onboarding)
    Dave->>Steve: new_round_start_prepared (signatures: forfeit/recycle/onboarding)
    Eve->>Steve: new_round_start_prepared (signatures: forfeit/recycle/onboarding)
    Victoria->>Steve: new_round_start_prepared (signatures: forfeit/recycle/liquidity)

    Note over Steve: Steve signs/broadcasts vtxo-root (on-chain) and announces round start
    Steve->>Alice: new_round_start (root deposited; round active)
    Steve->>Bob: new_round_start (root deposited; round active)
    Steve->>Carol: new_round_start (root deposited; round active)
    Steve->>Dave: new_round_start (root deposited; round active)
    Steve->>Eve: new_round_start (root deposited; round active)
    Steve->>Victoria: new_round_start (root deposited; round active)
```

1. Steve issues a `new_round_initiate`
    1. Informs everybody that a new round is starting.
2. Alice, Bob, Carol, Dave and Eve reply with `new_round_join`
    1. Here the Users declare what to onboard, what to offboard and what to transition.
3. Steve issues a `new_round_vtxo_tree_proposal`
    1. Here Steve sends out the VTXO tree for signing.
4. Alice, Bob, Carol, Dave and Eve issue a `new_round_vtxo_tree_accept`
    1. Here the Users return the signed nodes.
5. Steve issues a `new_round_prepare_start`
    1. Here Steve sends out the new root for signature, binding the tree to the root.
6. Alice, Bob, Carol, Dave and Eve issue a `new_round_start_prepared`.
    1. Here all Users accept the tree.
7. Steve issues a `new_round_start`.
    1. The root is deposited and the round starts. This is the official start of the round.

### Example 1.1 – Alice pays Bob

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates an HTLC based on H(P) and proposes a state update.
    1. Creates vtxo_spend_request, tagged as an HTLC_Offer.
    2. Steve signs vtxo_spend_complete.
4. Bob releases P.
    1. Creates vtxo_spend_request, tagged as a HTLC_Success.
    2. Steve signs vtxo_spend_complete.

### Example 1.2 – Round ends, Bob offboards 0.5 BTC, Dave is offline and misses the round, Eve onboards 0.4 BTC.

1. The block designated as the closing block at the start of the round is mined.
2. Steve issues a **new_round_initiate** message signaling a new round.
3. Steve issues **round_closed**. He tags the **new_round_initiate** message indicating the round to be transferred to.
4. The Users decide how to proceed with the new round, and they each issue a **new_round_join** message.
    1. Alice indicates that she wants to transfer all of her capital.
    2. Bob indicates that he wants to offboard 0.5 BTC.
    3. Carol indicates that she wants to transfer all of her capital.
    4. Dave is offline and does not send a message, hence missing the round. His capital is left in the closed round and will be recycled.
    5. Eve indicates that she wants to transfer all her current capital and also to onboard 0.4 BTC.
5. Steve collects the responses and, after the timeout is reached, issues a **new_round_vtxo_tree_proposal**. This includes:
    1. A new round transaction containing:
        1. A new VTXO root output, locked with A+B+C+E+S | S + T=2000.
        2. An offboarding output for Bob
        3. A forfeit root output.
        4. A funding input for Eve.
        5. A funding input for Steve.
    2. Forfeit control transaction.
    3. Forfeit transactions for the capital that is being transferred.
    4. A recycle transaction for the round containing:
        1. An output with Dave's capital based on his recycle address.
        2. An output to Steve with the rest of the capital.
6. The Users, Alice, Bob, Carol and Eve verify that the tree matches the request, and then each issues a **new_round_vtxo_tree_accept**. It contains:
    1. Signatures for the new VTXO tree.
7. Steve verifies that everything is there and then issues a **new_round_funding_request**.
8. The Users, Alice, Bob, Carol and Eve verify that everything is OK, and then each issues a **new_round_funding_accept**. It contains:
    1. Signatures for the forfeit transactions.
    2. Signatures for the funding transactions (only Eve has one).
    3. Signatures for the recycle transaction.
9. Steve verifies that the funding has been received, commits the transaction, and then issues a **new_round_start**.
10. Dave comes online, understands that he missed the round, and issues a **recycle_accept** containing his signature for the recycle transaction.
11. Steve submits the recycle transaction.

