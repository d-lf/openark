# OpenARK

### Design properties properties

1. HTLC compatible model (must be part of a lightning chain)
2. Support for RGB in ARK, define how the RGB model works in these channels.
3. Use nostr as transport protocoll.

### Expected outcome

1. A Bolt document explaining how lightning can be extended into a VTXO based multi-channel with a trusted party as a resolver (ARK-channle).
2. A NIP document that proposes a nostr based transport model for the messages.

## Roles

### Ark Service Provider
    Operates the nostrrelay and runs the ARK.

### Co Verifier
    Has one of the keys to S, aswell as there own user key. if a user is a coverifier can be hidden. 

### User
    Has only his own userkey. 

## Examples

### Example 1 - Alice, Bob, Carol, Dave and Eve create a VTXO tree with Steve acting as the Ark Service provider

![Architecture overview](./vtxo-tree.svg)

### Example 2 - Alice makes a cross atomic swap with Bob.

We start with a VTXO tree with two values, one value is regular bitcoin and a second value that is defined using RGB. At the start everybody has 1.0 och BTC and 1.0 och SecondCoin (SC). The goal is for Bob and Alice to be able to swap 0.2 BTC for 0.2 SC.

1. Alice creates a secret P
2. Alice creates a HTLC VTXO leaf (HTLC 1) where one of the outs is locked with H(P), the output is created in such a way that Bob can claim the VTXO either on-chain using a unilateral exit or in-ark if and only if he reveals P to S.
3. Alice sends the transaction to Bob (This shoud be a nostr-message that is public)
4. Bob verifies the transaction, and then issues a counter transaction using the same H(P), HTLC 2. 
5. Bob sends the transaction to Alice. (This should be a nostr-message that is public)
6. Alice requests to Service-provider to cosign the transition from HTLC 1 to VTXO A2, revealing P to Bob.
7. Service-provider grants Alice request, creating VTXO A2.
8. Bob requests to Service-provider to cosign the transaction from HTLC 2 to VTXO B2.
9. Service-provider grants Bobs request, creating VTXO B2.

### Example 3 - Alice fails a cross atomic swap with Bob, timeout triggered.

1. Alice creates a secret P
2. Alice creates a HTLC VTXO leaf (HTLC 1) where one of the outs is locked with H(P), the output is created in such a way that Bob can claim the VTXO either on-chain using a unilateral exit or in-ark if and only if he reveals P to S.
3. Alice sends the transaction to Bob (This shoud be a nostr-message that is public)
4. Bob verifies the transaction, and then issues a counter transaction using the same H(P), HTLC 2.
5. Bob sends the transaction to Alice. (This should be a nostr-message that is public)
6. Alice fails to request cosignature from Service-provider in time. Timeout is triggered.
8. Bob requests to Service-provider to cosign the transaction from HTLC 2 to VTXO B2.
9. Service-provider grants Bobs request, creating VTXO B2.
10. Alice requests to Service-provider to cosign the transition from HTLC 1 to VTXO A2.
11. Service-provider grants Alice request, creating VTXO A2.