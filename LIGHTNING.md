

### Alice Pays Bob, in lightning

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates HTLC based on H(P) and proposes a state updates the state.
   1. Alice sends update_add_htlc to Bob, this contains H(P)
4. Alice is finished with here proposed updates and initiates a state transition 
   1. Alice sends commitment_signed, signing BUE2.
   2. Bob sends revoke_and_ack, revoking BUE1.
   3. Bob sends commitment_signed, signing AUE2.
   4. Alice sends revoke_and_ack, revoking AUE1.
5. Bob releases P, and triggers a new proposal. 
   1. Bob sends update_fulfill_htlc
6. Bob is finished with his proposals and triggers a new state update.
   1. Bob sends commitment_signed, signing AUE3.
   2. Alice sends revoke_and_ack, revoking AUE2.
   3. Alice sends commitment_signed, signing BUE3.
   4. Bob sends revoke_and_ack, revoking BUE2.





