# I2P-Bote V3

## 1. Introduction

I2P-Bote is a serverless pseudonymous email exchange service for the I2P network. Emails are  
stored encrypted on other I2P-bote nodes.  
There is a SMTP/POP interface [TO BE IMPLEMENTED] to use with an email client, and a web interface  
that lets you change settings or send/read email.

Email can be sent through a number of other nodes (relays) [TO BE IMPLEMENTED] for increased security, or directly to  
a set of storage nodes for faster delivery. The same applies to retrieving email.  
All nodes are created equal. There are no "supernodes" or designated relay/storage nodes.  
Everybody acts as a potential relay and storage node. The maximum amount of disk space used for  
relayed/stored email packets can be configured by the user [TO BE IMPLEMENTED].  
Before an email is sent to a relay, it is broken up into packets and encrypted with the recipient's  
public key.

Email packets are stored redundantly in a distributed hash table (DHT). Stored email packets are  
kept for at least 100 days, during which the recipient can download them.  
For relays, the guaranteed retention time is only 7 days.  
If a node runs out of email storage space, and there are no old packets that can be deleted, the  
node refuses storage requests.

TODO only use names, not xxx@yyy.zzz style addresses. An @host can be used optionally for  
     email clients that insist on it, like Thunderbird, but the @host part is dropped by I2P-Bote.  
Email addresses are entered in the format "username@domain.i2p"; they must match an entry in the  
local address book or the distributed email directory. An exception are addresses of the form  
"dest_key@bote.i2p", where dest_key is a 516-byte base64-encoded email keypair. No lookup is done  
for these addresses; the dest_key part is directly used for routing the email.

Below is a diagram of how an email packet is routed from a sender to a recipient:

```
                    .-------.         .-------.         .--------.
                    | Relay |  ---->  | Relay |  ---->  | Storer |  --------------------.
                _   `-------'         `-------'         `--------'                       `\
                /`                                                                         |
               /                                                                           |
 .--------.   /      .-------.        .-------.         .--------.                         |
 | Sender |  ----->  | Relay |  --->  | Relay |  ---->  | Storer |  -------------.         |
 `--------'   \      `-------'        `-------'         `--------'                `\       |
               \                                                                    |      |
               _\/                                                                  |      |
                    .-------.         .-------.         .--------.                  |      |
                    | Relay |  ---->  | Relay |  ---->  | Storer |  ------.         |      |
                    `-------'         `-------'         `--------'         `\       |      |
                                                                             |      |      |
                                                                             V      V      V

                                           .--------------- Kademlia DHT -------------------------.
                                           |                                .---------.           |
                                           |   .---------.                  | Storage |           |
                                           |   | Storage |     .---------.  |  Node   |           |
                                           |   |  Node   |     | Storage |  `---------'           |
                                           |   `---------'     |  Node   |                        |
                                           |                   `---------'            .---------. |
                                           |  .---------.   .---------.               | Storage | |
                                           |  | Storage |   | Storage |  .---------.  |  Node   | |
                                           |  |  Node   |   |  Node   |  | Storage |  `---------' |
                                           |  `---------'   `---------'  |  Node   |              |
                                           |                             `---------'              |
                                           `------------------------------------------------------'

                                                                             |      |      |
                     .-------.          .-------.         .---------.       _'      |      |
                     | Relay |  <-----  | Relay |  <----  | Fetcher |  <---'        |      |
                  /  `-------'          `-------'         `---------'               |      |
                 /                                                                  |      |
               \/_                                                                  |      |
 .-----------.         .-------.        .-------.         .---------.              _'      |
 | Recipient |  <----  | Relay |  <---  | Relay |  <----  | Fetcher |  <----------'        |
 `-----------'  _      `-------'        `-------'         `---------'                      |
               |\                                                                          |
                 \                                                                         |
                  \  .-------.          .-------.         .---------.                     _'
                     | Relay |  <-----  | Relay |  <----  | Fetcher |  <-----------------'
                     `-------'          `-------'         `---------'
```

The number of relays for sending or retrieving an email is user-configurable.

TODO explain how an index packet is relayed back to the recipient, who then uses another chain to get the email packets  
referenced in the index packet.

For higher performance (but reduced anonymity), it is possible to use no relays, no storers/fetchers,  
i.e. sender and recipient talk to the storage nodes directly. If both sender and recipient chose not to  
use relays, the diagram looks like this:

```
 .--------.
 | Sender |  ----------------------------.
 `--------'                               `\
                                            |
                                            V

               .--------------- Kademlia DHT -------------------------.
               |                                .---------.           |
               |   .---------.                  | Storage |           |
               |   | Storage |     .---------.  |  Node   |           |
               |   |  Node   |     | Storage |  `---------'           |
               |   `---------'     |  Node   |                        |
               |                   `---------'            .---------. |
               |  .---------.   .---------.               | Storage | |
               |  | Storage |   | Storage |  .---------.  |  Node   | |
               |  |  Node   |   |  Node   |  | Storage |  `---------' |
               |  `---------'   `---------'  |  Node   |              |
               |                             `---------'              |
               `------------------------------------------------------'

                                            |
                                            |
 .-----------.                             _'
 | Recipient |  <-------------------------'
 `-----------'
```

The code is licensed under the GPL. The author can be reached at HungryHobo@mail.i2p, either in  
German or English.

## 2. Kademlia

TODO explain modified Kademlia: index packets, 2-stage retrieval;  
exhaustive Kademlia search for index packets, standard Kademlia search for email packets;  
no need for exponential expiration time (used in standard kademlia to prevent over-caching of popular files) because  
emails are only retrieved once (or a few times if there is a transmission error).  
Delete Authorization keys / Verification hashes.

Replication of email packets is done by index packet storage nodes.

TODO mention the problem of highly unbalanced trees and the sibling list solution.

## 3. Packet Types

As of release 0.2.4, the protocol version is 3. It is incompatible to earlier versions.

### 3.1. Data Packets

These are always sent wrapped in a Communication Packet (see 3.1).  
E denotes an encrypted field.  
Supported encryption and signature algorithms:  
  ALG=1: ElGamal-2048 / DSA-1024 / AES-256 / SHA-256 (same as the I2P router uses)  
  ALG=2: ECDH-256 / ECDSA-256 / AES-256 / SHA-256  
  ALG=3: ECDH-521 / ECDSA-521 / AES-256 / SHA-256  

```
   Packet Type         | Field | Data Type  | Description
  ---------------------+-------+------------+----------------------------------------------------
   Email Packet,       |       |            | An email or email fragment, 1 recipient. Part of
   encrypted           |       |            | the packet is encrypted with the recipient's key.
                       | TYPE  | 1 byte     | Value = 'E'
                       | VER   | 1 byte     | Protocol version
                       | KEY   | 32 bytes   | DHT key of the packet, SHA-256 hash of LEN+DATA
                       | TIM   | 4 bytes    | The time the packet was stored on a storage node
                       | DV    | 32 bytes   | Delete verification hash, SHA-256 hash of DA
                       | ALG   | 1 byte     | Encryption algorithm used
                       | LEN   | 2 bytes    | Length of DA+DATA together
                       | DA    | 32 bytes E | Delete authorization key, random data
                       | DATA  | byte[]   E | An Unencrypted Email Packet
  ---------------------+-------+------------+----------------------------------------------------
   Email Packet,       |       |            | Storage format for the Incomplete Email Folder.
   unencrypted         | TYPE  | 1 byte     | Value = 'U'
                       | VER   | 1 byte     | Protocol version
                       | MSID  | 32 bytes   | Message ID in binary format
                       | DA    | 32 bytes   | Delete authorization key, random data
                       | FRID  | 2 bytes    | Fragment Index of this packet (0..NFR-1)
                       | NFR   | 2 bytes    | Number of fragments in the email
                       | MLEN  | 2 bytes    | Length of the MSG field
                       | MSG   | byte[]     | email content (MLEN bytes)
  ---------------------+-------+------------+----------------------------------------------------
   Index Packet        |       |            | Contains the DHT keys of one or more Email Packets,
                       |       |            | and a Delete verification hash and time stamp for
                       |       |            | each DHT key.
                       | TYPE  | 1 byte     | Value = 'I'
                       | VER   | 1 byte     | Protocol version
                       | DH    | 32 bytes   | SHA-256 hash of the recipient's email destination
                       | NP    | 4 bytes    | Number of entries in the packet
                       | KEY1  | 32 bytes   | DHT key of the first Email Packet
                       | DV1   | 32 bytes   | Delete verification hash for KEY1, SHA-256 hash of DA in the email packet
                       | TIM1  | 4 bytes    | The time the key KEY1 was added
                       | KEY2  | 32 bytes   | DHT key of the second Email Packet
                       | DV2   | 32 bytes   | Delete verification hash for KEY2, SHA-256 hash of DA in the email packet
                       | TIM2  | 4 bytes    | The time the key KEY2 was added
                       | ...   | ...        | ...
                       | KEYn  | 32 bytes   | DHT key of the n-th Email Packet
                       | DVn   | 32 bytes   | Delete verification hash for KEYn, SHA-256 hash of DA in the email packet
                       | TIMn  | 4 bytes    | The time the key KEYn was added
  ---------------------+-------+------------+----------------------------------------------------
   Deletion Info       |       |            | Contains information about deleted DHT items, which
   Packet              |       |            | can be Email Packets or Index Packet entries.
                       |       |            | Only used locally.
                       | TYPE  | 1 byte     | Value = 'T'
                       | VER   | 1 byte     | Protocol version
                       | NP    | 4 bytes    | Number of entries in the packet
                       | KEY1  | 32 bytes   | First DHT key
                       | DA1   | 32 bytes   | Delete Authorization for KEY1
                       | TIM1  | 4 bytes    | The time the key KEY1 was added
                       | KEY2  | 32 bytes   | Second DHT key
                       | DA2   | 32 bytes   | Delete Authorization for KEY2
                       | TIM2  | 4 bytes    | The time the key KEY2 was added
                       | ...   | ...        | ...
                       | KEYn  | 32 bytes   | The n-th DHT key
                       | DAn   | 32 bytes   | Delete Authorization for KEYn
                       | TIMn  | 4 bytes    | The time the key KEYn was added
  ---------------------+-------+------------+----------------------------------------------------
   Relay Data Packet   |       |            | Contains an I2P destination, a time window,
                       |       |            | and a Relay Request
                       | TYPE  | 1 byte     | Value = 'L'
                       | VER   | 1 byte     | Protocol version
                       | DMIN  | 4 bytes    | Minimum delay in seconds
                       | DMAX  | 4 bytes    | Maximum delay in seconds (no guarantee!)
                       | NEXT  | 384 bytes  | Destination to forward the packet to
                       | DLEN  | 2 bytes    | Length of DATA field in bytes
                       | DATA  | byte[]     | A Relay Request
  ---------------------+-------+------------+-------------------------------------------------
   Peer List           |       |            | Response to a Find Close Peers Request or
                       |       |            | a Peer List Request
                       | TYPE  | 1 byte     | Value = 'P'
                       | VER   | 1 byte     | Protocol version
                       | NUMP  | 2 bytes    | Number of peers in the list
                       | P1    | 384 bytes  | Destination key
                       | P2    | 384 bytes  | Destination key
                       | ...   | ...        | ...
                       | Pn    | 384 bytes  | Destination key
  ---------------------+-------+------------+-------------------------------------------------
   Email Destination   |       |            | TBD
                       | TYPE  | 1 byte     | Value = 'M'
                       |       |            | 
                       |       |            | 
                       |       |            | 
                       |       |            | 
  ---------------------+-------+------------+-------------------------------------------------
```

### 3.2. Communication Packets

Communication packets are used for sending data between two I2P-Bote nodes. They contain a  
data packet, see 3.1.  
All Communication Packets start with a four-byte prefix, followed by one byte for the packet  
type and one byte for the packet version. Some have a 32-byte Request ID after the packet  
version.  
Strings in packets are UTF8 encoded.

```
   Packet Type         | Field | Data Type  | Description
  ---------------------+-------+------------+----------------------------------------------------
   Relay Request       |       |            | A packet that tells the receiver to do communicate
                       |       |            | with a peer, or peers, on behalf of the sender.
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'Y'
                       | VER   | 1 byte     | Protocol version
                       | HLEN  | 2 bytes    | HashCash length
                       | HK    | byte[]     | HashCash token (HLEN bytes)
                       | DLEN  | 2 bytes    | Length of DATA field in bytes
                       | DATA  | byte[]   E | Encrypted with the recipient's public key. Can
                       |       |            | contain an Email Packet, an Index Packet, or a
                       |       |            | Relay Data Packet.
  ---------------------+-------+------------+----------------------------------------------------
   Fetch Request       |       |            | Request to a chain end point to fetch emails
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'F'
                       | VER   | 1 byte     | Protocol version
                       | SID   | 32 bytes   | Request ID, used for responses
                       | DTYP  | 1 byte     | Type of data to retrieve:
                       | KEY   | 32 bytes   | DHT key to look up
                       | KPR   | 384 bytes  | Email keypair of recipient
                       | RLEN  | 2 bytes    | Length of RET field
                       | RET   | byte[]     | Relay packet, contains the return chain (RLEN bytes)
  ---------------------+-------+------------+----------------------------------------------------
   Response Packet     |       |            | Response to a Retrieve Request, Fetch Request,
                       |       |            | Find Close Peers Request, or a Peer List Request
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'R'
                       | VER   | 1 byte     | Protocol version
                       | SID   | 32 bytes   | Request ID of the request packet
                       | STA   | 1 byte     | Status code:
                       |       |            |   0 = OK
                       |       |            |   1 = General error
                       |       |            |   2 = No data found
                       |       |            |   3 = Invalid packet
                       |       |            |   4 = Invalid HashCash
                       |       |            |   5 = Not enough HashCash provided
                       |       |            |   6 = No disk space left
                       | DLEN  | 2 bytes    | Length of DATA field; can be 0 if no payload
                       | DATA  | byte[]     | A Data Packet
  ---------------------+-------+------------+----------------------------------------------------
   Peer List Request   |       |            | A request for a list of high-reachability relay peers
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'A'
                       | VER   | 1 byte     | Protocol version
                       | SID   | 32 bytes   | Request ID, used for responses
  ---------------------+-------+------------+----------------------------------------------------
```

#### DHT Communication Packets

```
   Packet Type         | Field | Data Type  | Description
  ---------------------+-------+------------+----------------------------------------------------
   Retrieve Request    |       |            | DHT Request for a value for a key
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'Q'
                       | VER   | 1 byte     | Protocol version
                       | SID   | 32 bytes   | Request ID, used for responses
                       | DTYP  | 1 byte     | Type of data to retrieve:
                       |       |            |   'I' = Index Packet
                       |       |            |   'E' = Email Packet
                       |       |            |   'M' = Email destination
                       | KEY   | 32 bytes   | DHT key to look up
  ---------------------+-------+------------+----------------------------------------------------
   Store Request       |       |            | DHT Store Request
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'S'
                       | VER   | 1 byte     | Protocol version
                       | SID   | 32 bytes   | Request ID, used for responses
                       | HLEN  | 2 bytes    | HashCash length
                       | HK    | byte[]     | HashCash token (HLEN bytes)
                       | DLEN  | 2 bytes    | Length of DATA field
                       | DATA  | byte[]     | Data packet to store (DLEN bytes). Can be an
                       |       |            | Index Pkt / Email Pkt / Email Destination)
  ---------------------+-------+------------+----------------------------------------------------
   Email Packet        |       |            | Request to delete an Email Packet by DHT key
   Delete Request      |       |            | 
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'D'
                       | VER   | 1 byte     | Protocol version
                       | KEY   | 32 bytes   | DHT key of the Email Packet to delete
                       | DA    | 32 bytes   | Delete Authorization, SHA-256 must = DV in the email pkt
  ---------------------+-------+------------+----------------------------------------------------
   Index Packet        |       |            | Request to remove one or more entries (Email
   Delete Request      |       |            | Packet keys) from an Index Packet
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'X'
                       | VER   | 1 byte     | Protocol version
                       | DH    | 32 bytes   | The Email Destination hash of the Index Packet
                       | N     | 1 byte     | Number of entries in the packet
                       | DHT1  | 32 bytes   | First DHT key to remove
                       | DA1   | 32 bytes   | Delete Authorization, SHA-256 must = DV1 in the email pkt
                       | DHT2  | 32 bytes   | Second DHT key to remove
                       | DA2   | 32 bytes   | Delete Authorization for DHT2, random data
                       | ...   | ...        | ...
                       | DHTn  | 32 bytes   | n-th DHT key to remove
                       | DAn   | 32 bytes   | Delete Authorization for DHTn, random data
  ---------------------+-------+------------+----------------------------------------------------
   Find Close Peers    |       |            | Request for k peers close to a key
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'C'
                       | VER   | 1 byte     | Protocol version
                       | SID   | 32 bytes   | Request ID, used for responses
                       | KEY   | 32 bytes   | DHT key
  ---------------------+-------+------------+----------------------------------------------------
```

## 4. Protocols For Inter-Node Communication

### 4.1. Sending Email via relays

I2P nodes involved: A=Sender, R1...Rn=Relays, S1...Sm=Storage Nodes

1. A sends an Relay Packet to R1.
2. R1 decrypts the packet, waits a random amount of time, and sends it to R2.
3. R2 confirms delivery with R1.
4. Repeat until packet arrives at Rn.
5. Rn decrypts the Relay Packet into an Email Packet.
6. Rn sends the packet to S1,...,Sm through a Kademlia STORE

### 4.2. Retrieving Email via relays (not implemented yet)

I2P nodes involved: A=Address Owner, O1...On=Outbound Relays, I1...In=Inbound Relays, S1...Sm=Storage Nodes

1.  A sends a Relay Packet to O1.
2.  O1 decrypts the packet, waits a random amount of time, and sends it to O2.
3.  O2 confirms delivery with O1.
4.  Repeat until packet arrives at On.
5.  On decrypts the Relay Packet into a Retrieve Request.
6.  On does a Kademlia LOOKUP for the email address.
7.  Nodes S1,...,Sm verify the expiration time, request number, and A's signature on the Retrieve Request.
8.  Nodes S1,...,Sm send their stored Email Packets to Rn.
9.  On confirms delivery with S1,...,Sm
10. On makes a Relay Packet from the Email Packet and sends it to I1.
11. I1 confirms delivery with On.
12. I1 sends each packet to I2.
13. I2 confirms delivery with I1.
14. Repeat the last two steps until packet arrives at A.
15. A decrypts the packet into a plaintext email or email fragment.

### 4.3. Pinging a node

### 4.4. Responding to a Ping

### 4.5. Storing an Email Packet on a storage node

### 4.6. Sending a Retrieve Request to a Node

### 4.7. Deleting an Email Packet

Every time a recipient receives an email packet (from one or more storage nodes), it asks the storage  
node(s) to delete the packet.
  
[TODO: This doesn't work for relayed email packets because the recipient doesn't know the storage nodes.  
      Maybe include the storage node in an email packet? Or relay the delete request and have the  
      relay endpoint do a findClosestNodes + delete? Or just delete the index packet entry and let  
      replication take care of deleting the email packets?]

It also sends a delete request to the nodes storing the index packet, asking them to delete the DHT key  
of the email packet. Each index packet node then removes the DHT key from the stored index packet, and  
adds the DHT key to a list of deleted keys it maintains for each index packet (DHT key + deletion key,  
actually). The purpose of this is so a node that is about to replicate an email packet can find out if  
it missed an earlier delete request for that packet, in which case the node "replicates" the delete  
request rather than the packet itself. This helps reduce storage space by removing old Email Packets  
from nodes that weren't online at the time the delete request was sent initially.
  
1. Recipient knows the deletion key after it decrypts the email packet (the packet also contains a  
   "plaintext deletion key" field, which is all zeros after it leaves a storage node).
2. Recipient sends a Delete Request to all nodes that responded to the query for the Email Destination,  
   asking them to mark the Email Packet's DHT key "deleted".  
***** TODO deletion key for index packets? use signed del requests instead? *****
3. Recipient sends a Delete Request to all peers that responded to the query for the Email Packet

### 4.8. Replication of Email Packets

### 4.9. Replication of Index Packets

### 4.10. Looking up an email address in the directory

### 4.11. Announcing a new email address to the directory

### 4.12. Updating or deleting an email address from the directory

## 5. Algorithms Used By Nodes Locally

### 5.1. Sending Email   *** OUTDATED ***
  
1. Given a MIME email, GZip it, sign it, store it in the local outbox, split it into 30kB  
   packets plus an Index Packet, pad the packets if necessary, and store them locally.
2. Pick a random packet/recipient combination for which the wait time is up, and for which  
   less than <storage redundancy> distinct delivery confirmations have been received within the  
   timeout period, and the maximum number of retries has not been reached.
3. Set a random Request ID on the packet.
4. Encrypt the packet with the public key for the recipient
5. Encrypt the packet with the public keys of relay n, relay n-1, ..., relay 1, and add  
   the destination key of the next relay each time. Also included is a minimum/maximum delay  
   value for each relay.
6. Queue the packet for sending to the first relay.
7. Repeat from step 2 until no more packets left.
8. When all packets have been sent, move the email from the outbox to the sent folder.
  
Rationale for storing the whole email, not the packets, on disk: When sending to multiple  
recipients, only one copy of the email needs to be stored.

### 5.2. Retrieving Email

see also http://forum.i2p/viewtopic.php?p=19927#19927 ff.
   
1.  Randomly choose a set of n relay nodes,  R1...Rn (outgoing chain for request)
2.  Randomly choose a set of m relay nodes, S1...Sm.(return chain / inbound mail chain)  
      - with Sm being the node closest to the receiver  
        and S1 the chain node closest to fetcher
3.  Generate m+1 random 32-byte arrays (for XORing the return packets), X1...Xm+1
4.  With Sm's public key, encrypt the local destination key and Xm+1
5.  With Sm-1's public key, encrypt Sm's destination key, Xm, and the data from step 4)
6.  With Sm-2's public key, encrypt Sm-1's destination key, Xm-1, and the data from step 5)
7.  Repeat until the entire return chain, S1...Sm, has been onion-encrypted,  
      and also include a minimum/maximum delay value for each relay.
8.  Add S1's destination key and X1 to the data from step 7)
9.  Add the data (which is a Relay Packet) to a Retrieve Request packet.
10. Encrypt the resulting packet, using the public keys of relays  
    Rn (Fetcher), Rn-1, ..., R1, and add the destination key of the next relay each time.  
    Also included is a minimum/maximum delay value for each relay.
11. Set a random Request ID on the packet at each layer
12. Queue the packet for immediate sending to the first relay.
  
The following is done continuously and asynchronously, regardless of whether there a Retrieve
Request has been sent recently:
  
1. Nodes are always ready for incoming Email Packets.
2. For every email packet that is received, a Delete Request is sent out to the Kademlia network  
   via a relay chain.
  
### 5.3. Fetching Email Packets For an Address Through Kademlia

1. Do a Kademlia lookup for the keypair in the Retrieve Request
2. Extract Request IDs from the Index Packet(s)
3. Do a Kademlia lookup for each Request ID
4. Check the Request ID on the Email Packets and send them back to the first return relay
  
### 5.4. Relaying a Packet

1. Decrypt packet
2. Verify signature
3. XOR payload with the key included in the packet.
4. Read wait time range from packet
5. Wait for a random amount of time within the wait time range.
6. Queue packet for sending with that delay.
7. If no delivery confirmation within timeout period, wait for $resend_delay and repeat from step 5

### 5.5. Storing an Email Packet

1. If packet can't fit in storage space, delete stored email packets that are more than 100 days old.
2. If still no room, send back negative response
3. Otherwise, write packet to file and send positive response

## 6. Local Address Book

## 7. Distributed Email Directory

Each adress-keypair mapping is stored at multiple nodes. Nodes can only add an address to the  
distributed address directory, but they cannot edit or delete them. An address must be renewed  
periodically, or it expires after a year and is deleted from the address directory.  
A node only stores one keypair per email address. Whichever came first, stays in the address  
directory.  
When doing an address lookup, a error message is displayed to the user if lookup results from two  
nodes are different.  
It is important for the user to understand that using the distributed email directory is not as  
secure as manually adding keypairs to the local address book, although it is more convenient.

## 8. UI

### 8.1. Retrieving email

* Wait 20 seconds for any emails to arrive, then return from HTTP request or POP3 request.  
  All other emails will be received in the background.
* The "high priority" setting found in most email clients sets all relaying delays to zero.

## 9. Files

### 9.1. Identities file

Stores all email identities the local user has created. One line per entry.  
Format: <base64 key><tab><public name>[<tab><description>][<tab><email address>]

## 10. Glossary of Terms

### Email Destination
As the name implies, an Email Destination is an identifier by which somebody can be reached via I2P-Bote.  
An Email Destination is a Base64 string containing a public encryption key and a signature verification key.  
Example:

```  
uQtdwFHqbWHGyxZN8wChjWbCcgWrKuoBRNoziEpE8XDt8koHdJiskYXeUyq7JmpG
In8WKXY5LNue~62IXeZ-ppUYDdqi5V~9BZrcbpvgb5tjuu3ZRtHq9Vn6T9hOO1fa
FYZbK-FqHRiKm~lewFjSmfbBf1e6Fb~FLwQqUBTMtKYrRdO1d3xVIm2XXK83k1Da
-nufGASLaHJfsEkwMMDngg8uqRQmoj0THJb6vRfXzRw4qR5a0nj6dodeBfl2NgL9
HfOLInwrD67haJqjFJ8r~vVyOxRDJYFE8~f9b7k3N0YeyUK4RJSoiPXtTBLQ2RFQ
gOaKg4CuKHE0KCigBRU-Fhhc4weUzyU-g~rbTc2SWPlfvZ6n0voSvhvkZI9V52X3
SptDXk3fAEcwnC7lZzza6RNHurSMDMyOTmppAVz6BD8PB4o4RuWq7MQcnF9znElp
HX3Q10QdV3omVZJDNPxo-Wf~CpEd88C9ga4pS~QGIHSWtMPLFazeGeSHCnPzIRYD
```

### Email Address

Email Addresses in I2P-Bote are shortcuts for Email Destinations.  
Email Address <--> Email Destination mappings are stored in two places: the local address book and  
the distributed address directory.

### Email Identity

An Email Identity is an Email Destination plus a name you want to be known as when you use that identity.  
Technically speaking, an Email Identity consists of four things:

  * An Email Destination (i.e. two public keys)
  * The two private keys for the Email Destination
  * A public name which is shown to other people in emails
  * A description which is not shown to anybody but you.  
    It helps you remember which Email Identity you use for which purpose.

An email identity is not required for sending emails (although only "Anonymous" can be selected for the  
"sender" field).  

### I2P destination
The address of a I2P-Bote node on the I2P network. There is no need to know it unless you  
are having problems connecting to other I2P-Bote nodes.  
I2P destinations and Email Destinations look similar, but are completely independent of each other.

## 11. To Do / Ideas

* ignore store requests for existing packets so malicious index nodes cannot replace email packets on other index nodes
* instead of republishing after T seconds, pick a random time from the interval [T-a, T+a];  
  the probability should be low at T-a and then increase
* Cache email dest - closest node mappings for faster Kademlia lookups
* Seedless for bootstrapping
* some synchronized ArrayLists should be replaced with externally synchronized ArrayLists
* identities: "publish to distributed address book" button, asks for an email address
* b32 email destinations in the distributed address book? could also be useful for getting around the address size limit in POP/SMTP
* <jsp:useBean> instead of zero-param JSP functions?
* make sure del key is zeroed out when responding to a retrieve request for an EncryptedEmailPacket
* don't send delete requests to self, just delete the file locally
* button to delete all packets with an invalid protocol version
* auto-refresh network.jsp
* show remaining time for the 3-min delay at startup
* Replication per the Kademlia spec
* Have an option to disable automatic lookups in the distributed address directory for your own address
* Automatically create a different sender address for each recipient
* test relays periodically
* test storage nodes periodically  
* optionally pad all packets to a system-wide fixed size
* Always use the same amount of bandwidth, testing nodes or sending dummy messages  
  if necessary.
* Always keep incoming bandwidth the same as outgoing bandwidth
* Pad packets before encrypting to prevent tagging attacks
* onion packets/regular packets
* delivery confirmation to sender
* What about incoming bandwith, can it be limited?
* Turnkey outproxy feature?
* All nodes keep track of relay peers through Peer List Requests and by pinging known peers  
  periodically.
* select different set of hops for each packet
* instead of sending dummy messages for cover traffic, test gateways by routing packet to self
* keep track of uptimes + reliability of paths (or gateways), have a "minimum 
  uptime/min reliability percentage" setting for selecting gateways
* Use more storage nodes if email destination is known to be heavily frequented (like Kademlia does for popular hashes)
* mention that min/max delay = earliest/latest send time (explain that there is no guarantee for the send time being between min and max, it can be after max)
* On first run, offer to create a new email identity, and optionally assign an email address to it.
* Folder structure on disk should reflect folders in app. All email folders visible to the user have the same root directory: [app dir]/folders.
* ask user for confirmation before deleting an email identity
* have a view of the incomplete folder, so users can look at incomplete emails
* use a key store for email identities so they are not locally readable?
* unit test candidates:
    * KBucket
    * KademliaDHT
    * Identities
    * OutboxProcessor
* prioritize packets in I2PSendQueue? Should PeerListRequests get higher priority than relay packets etc.?
* AAAAA.... in local_dest.key file doesn't look right, fix
* Don't use Kademlia for getRandomNodes(), implement a RelayPeerManager instead
* don't use byte arrays for content at all, use InputStream instead so we don't have to load large attachments into memory
* ByteArrayOutputStreams don't need to be closed
* remove setFileName/getFileName from Email, move debug code that uses it into Outbox
* use I2PSocketManager.ping() and rename PING/PONG packets to REQ/RESP?
