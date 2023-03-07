# Протоколы (ЧЕРНОВИК)

## 1. Сохранение DHT-элемента через реле

Вовлечённые I2P узлы:

* `A` - Sender
* `R1...Rn` - Relays
* `S1...Sm` - Storage Nodes

Процесс:

1. A onion-encrypts the Store Request with the public keys of all hops, resulting in a Relay Packet.
2. A sends the Relay Packet to R1.
3. R1 decrypts the Packet, waits a random amount of time, and sends it to R2.
4. R2 confirms delivery with R1.
5. Repeat until Packet arrives at Rn.
6. Rn decrypts the Relay Packet into an Email Packet.
7. Rn sends the Packet to S1,...,Sm through a Kademlia STORE

## 2. Цепь возврата

In order to make it impossible for two non-adjacent nodes on the return chain to identify Relay Return Request packets, the payload_ and the entire Return Chain are re-encrypted at every hop.   
A new Correlation ID is generated at every hop.

Here is a simplified diagram of how a Relay Return Packet is sent from one hop to the next:

1. The Packet is received. HOP1 through HOP3 contain encrypted data (encrypted with the public key for the receiving I2P node):
```
      
      +------------------- Relay Return Request --------------------------+
      |                                                                   |
      |                                .-----KEY1-----.                   |
      |                .---KEY1---.   |  .---KEY2---.  |                  |
      |     .-KEY1-.  |  .-KEY2-.  |  | |  .-KEY3-.  | |  .---------.     |
      |     | HOP1 |  |  | HOP2 |  |  | |  | HOP3 |  | |  | PAYLOAD |     |
      |     `------’  |  `------’  |  | |  `------’  | |  `---------’     |
      |                `----------’   |  `----------’  |                  |
      |                                `--------------’                   |
      |                                                                   |
      +-------------------------------------------------------------------+
      
      KEYx denotes a layer of encryption.
```
2. The receiving node decrypts HOP1, HOP2, and HOP3. HOP1 is now in plain text. It contains thedestination of the next hop and an AES key.
```  
      +------------------- Relay Return Request --------------------------+
      |                                                                   |
      |                                .-----KEY2-----.                   |
      |                .---KEY2---.   |  .---KEY3---.  |                  |
      |     .------.  |  .------.  |  | |  .------.  | |  .---------.     |
      |     | HOP1 |  |  | HOP2 |  |  | |  | HOP3 |  | |  | PAYLOAD |     |
      |     `------’  |  `------’  |  | |  `------’  | |  `---------’     |
      |                `----------’   |  `----------’  |                  |
      |                                `--------------’                   |
      |                                                                   |
      +-------------------------------------------------------------------+
```
3. HOP1 is replaced with random data and moved to the end of the chain. The payload is encrypted with the AES key from HOP1:
```
      +------------------- Relay Return Request --------------------------+
      |                                                                   |
      |                      .-----KEY2-----.                             |
      |      .---KEY2---.   |  .---KEY3---.  |                            |
      |     |  .------.  |  | |  .------.  | |  .------.  .---AES---.     |
      |     |  | HOP3 |  |  | |  | HOP2 |  | |  | HOP1 |  | PAYLOAD |     |
      |     |  `------’  |  | |  `------’  | |  `------’  `---------’     |
      |      `----------’   |  `----------’  |                            |
      |                      `--------------’                             |
      |                                                                   |
      +-------------------------------------------------------------------+
```
4. The Packet is sent to the next node in the chain.

## 3. Retrieving DHT items via relays (not implemented yet)

I2P nodes involved:

- `A`=Address Owner
- `F`=Fetcher
- `O1...On`=Outbound Relays
- `I1...In`=Inbound Relays,
- `S1...Sm`=Storage Nodes

Process:

1.  A builds a Relay Request containing a Retrieve Request and sends it to F via O1, O2, ..., On as described in 4.1
2.  F queries the DHT for the DHT item
3.  F encrypts the DHT data with the public key for hop 1 of the return chain
4.  F moves the (RL1,RET1) block of the return chain to the end (behind RETn+1)
5.  F sends the results of steps 3) and 4) to I1
6.  I1 decrypts all (RLx,RETx) blocks of the return chain and finds I2 and the AES key in the first block
7.  I1 replaces (RL1,RET1) with random data that is encrypted with the PK for I2
8.  I1 moves the (RL1,RET1) block of the return chain to the end (behind RETn+1)
9.  I1 encrypts the payload_ with the AES key
10. I1 sends the results of steps 7) and 8) to I2
11. Repeat from 6) for I2, I3, ..., until the data reaches A (when the first byte of RET1 is non-zero)
12. A decrypts the (RL1,RET1) block which contains all AES keys
13. A decrypts the payload_ with the AES keys, starting from the last hop's key
14. The decrypted data contains the DHT data Packet

## 4. Deleting an Email Packet

Every time a recipient receives an email Packet directly from one or more storage nodes, it asks the storage node(s) to delete the Packet.   
If the email Packet is received through one or more relays, the recipient issues a delete request to a (possibly different) relay endpoint, which entails an additional findClosestNodes lookup because the endpoint has to find out which nodes are storing the Packet.   
  
The recipient also sends a delete request for index Packet, asking the storage nodes to delete the DHT key of the email Packet.   
Each index Packet node then removes the DHT key from the stored index Packet, and adds the DHT key to a list of deleted DHT keys it maintains for each Index Packet (plus the delete verification hash).   
The purpose of this is so a node that is about to replicate an email Packet can find out if it missed an earlier delete request for that Packet, in which case the node "replicates" the delete request rather than the Packet itself.   
This helps reduce storage space by removing old Email Packets from nodes that weren't online at the time the delete request was sent initially.
  
## 5 Deleting an Index Packet Entry

Similar to deleting an email Packet.

## 6. Replication

See comments at the beginning of src/i2p/bote/network/kademlia/ReplicateThread.java

Replicates locally stored DHT data on startup and every REPLICATE_INTERVAL seconds after that.

The basic algorithm goes like this:

* Do a lookup for the local node ID to refresh the s-bucket
* For each locally stored DHT item (index packet entries and email packets) that has not been deleted:
  * Store the entry on the k closest nodes, based on info from local buckets
  * If at least one peer responds with a valid delete request,
    * delete it locally (this is already handled by code outside this class), and
    * send a delete request to the nodes that didn't respond (which they won't if they don't know the packet has been deleted)
* Otherwise, replication for that entry is finished.

## 7. Retrieving Email via relays

See also http://forum.i2p/viewtopic.php?p=19927#19927

Process:

1. Randomly choose a set of n relay nodes,  R1...Rn (outgoing chain for request)
2. Randomly choose a set of m relay nodes, S1...Sm. (return chain / inbound mail chain)

  - with Sm being the node closest to the receiver and S1 the chain node closest to fetcher

3. Generate m+1 random 32-byte arrays (for XORing the return packets), X1...Xm+1
    (This is done so the outgoing chain's endpoint or inbound first hop need not know all inbound hops in order to perform hop-to-hop encryption. Each hop will decrypt one 32-byte random array and encrypt the Packet with it, so it looks different at each hop without one node at the beginning knowing all hops)
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
