# 5. Algorithms Used By Nodes Locally

## 5.1. Retrieving Email via relays

See also http://forum.i2p/viewtopic.php?p=19927#19927 ff.
   
1. Randomly choose a set of n relay nodes,  R1...Rn (outgoing chain for request)
2. Randomly choose a set of m relay nodes, S1...Sm. (return chain / inbound mail chain)

  - with Sm being the node closest to the receiver and S1 the chain node closest to fetcher

3. Generate m+1 random 32-byte arrays (for XORing the return packets), X1...Xm+1
    (This is done so the outgoing chain's endpoint or inbound first hop need
    not know all inbound hops in order to perform hop-to-hop encryption. Each
    hop will decrypt one 32-byte random array and encrypt the Packet with it,
    so it looks different at each hop without one node at the beginning
    knowing all hops)
4. With Sm's public key, encrypt the local destination key and Xm+1
5. With Sm-1's public key, encrypt Sm's destination key, Xm, and the data from step 4)
6. With Sm-2's public key, encrypt Sm-1's destination key, Xm-1, and the data from step 5)
7. Repeat until the entire return chain, S1...Sm, has been onion-encrypted, and also include a delay for each hop.
8. Add S1's destination key and X1 to the data from step 7)
9. Add the data (which is a Relay Packet) to a Retrieve Request Packet.
10. Encrypt the resulting Packet, using the public keys of relays Rn (Fetcher), Rn-1, ..., R1, and add the destination key of the next relay each time. Also included is a delay for each relay.
11. Set a random Correlation ID on the Packet at each layer
12. Add the Packet to the relay Packet folder.
13. When a reply Packet is received, decrypt it first with the local node's privatedecryption key, then with all the random keys X1...Xm+1.
