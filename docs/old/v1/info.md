# I2P-Bote V1

## 1. Introduction

I2P-Bote is a serverless pseudonymous email exchange service for the I2P network. Emails are  
stored encrypted on other I2P-bote nodes.  
There is a SMTP/POP interface [TO BE IMPLEMENTED] to use with an email client, and a web interface  
[TO BE IMPLEMENTED] that lets you change settings or send/read email.

Email can be sent through a number of other nodes (relays) for increased security, or directly to  
a set of storage nodes for faster delivery. The same applies to retrieving email.  
All nodes are created equal. There are no "supernodes" or designated relay/storage nodes.  
Everybody acts as a potential relay and storage node. The maximum amount of disk space used for  
relayed/stored email packets can be configured by the user.  
Before an email is sent to a relay, it is broken up into packets and encrypted with the recipient's  
public key.

Email packets are stored redundantly in a distributed hash table (DHT). Stored email packets are  
kept for at least 100 days, during which the recipient can download them.  
For relays, the guaranteed retention time is only 7 days.  
If a node runs out of email storage space, and there are no old packets that can be deleted, the  
node refuses storage requests.

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

The number of relays for sending or retrieving an email is user-configurable. For higher performance (but  
reduced anonymity), it is possible to use no relays, no storers/fetchers, i.e. sender and recipient  
talk to the storage nodes directly.

TODO explain how an index packet is relayed back to the recipient, who then uses another chain to get the email packets  
referenced in the index packet.

The code is licensed under the GPL. The author can be reached at HungryHobo@mail.i2p, either in  
German or English.


## 2. Kademlia

TODO explain modified Kademlia: index packets, 2-stage retrieval;  
exhaustive Kademlia search for index packets, standard Kademlia search for email packets;  
no need for exponential expiration time (used in standard kademlia to prevent over-caching of popular files) because  
emails are only retrieved once (or a few times if there is a transmission error).  

TODO mention the problem of highly unbalanced trees and the sibling list solution.


## 3. Packet Types

### 3.1. Communication Packets

Communication packets are used for sending data between two I2P-Bote nodes. They contain a  
payload packet, see 3.2.  
All Communication Packets start with a four-byte prefix, followed by one byte for the packet  
type, one byte for the packet version, and a 32-byte packet ID.  
Strings are UTF8 encoded.

```
   Packet Type         | Field | Data Type  | Description
  ---------------------+-------+------------+----------------------------------------------------
   Relay Request       |       |            | Request to a node to forward a Relay Packet
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'Y'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | PID   | 32 bytes   | Packet ID, used for delivery confirmation
                       | HLEN  | 2 bytes    | HashCash length
                       | HK    | byte[]     | HashCash token (HLEN bytes)
                       | DLEN  | 2 bytes    | Length of DATA field in bytes, up to ~30k
                       | DATA  | byte[]     | A Relay Packet
  ---------------------+-------+------------+----------------------------------------------------
   Fetch Request       |       |            | Request to a chain end point to fetch emails
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'T'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | PID   | 32 bytes   | Packet ID, used for delivery confirmation
                       | DTYP  | 1 byte     | Type of data to retrieve:
                       | KEY   | 32 bytes   | DHT key to look up
                       | KPR   | 384 bytes  | Email keypair of recipient
                       | RLEN  | 2 bytes    | Length of RET field
                       | RET   | byte[]     | Relay packet, contains the return chain (RLEN bytes)
  ---------------------+-------+------------+----------------------------------------------------
   Response Packet     |       |            | Response to a Retrieve Request, Fetch Request,
                       |       |            | Find Close Peers Request, or a Relay List Request
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'N'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | PID   | 32 bytes   | Packet Id of the request packet
                       | STA   | 1 byte     | Status code:
                       |       |            |   0 = OK
                       |       |            |   1 = General error
                       |       |            |   2 = No data found
                       |       |            |   3 = Invalid packet
                       |       |            |   4 = Invalid HashCash
                       |       |            |   5 = Not enough HashCash provided
                       |       |            |   6 = No disk space left
                       | DLEN  | 2 bytes    | Length of DATA field; can be 0 if no payload
                       | DATA  | byte[]     | A Payload Packet
  ---------------------+-------+------------+----------------------------------------------------
   Delete Request      |       |            | Request to delete all copies of a packet by DHT key
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'D'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | KEY   | 32 bytes   | DHT key of the packet that is to be deleted
                       | DEL   | 32 bytes   | Deletion key, encrypts to DEL value in the email pkt
  ---------------------+-------+------------+-------------------------------------------------
   Ping                |       |            | Check to see if a host is still up.
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'P'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | PID   | 32 bytes   | Packet ID
  ---------------------+-------+------------+-------------------------------------------------
   Pong                |       |            | Let the the pinger know we're still there.
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'O'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | PID   | 32 bytes   | Packet ID of the corresponding Ping packet
                       | HKS   | 1 byte     | Minimum HashCash strength this node will accept
  ---------------------+-------+------------+----------------------------------------------------
   Relay List Request  |       |            | "Send me your list of relay peers"
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'A'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | PID   | 32 bytes   | Packet ID
  ---------------------+-------+------------+----------------------------------------------------
```

#### DHT Packets

```  
   Packet Type         | Field | Data Type  | Description
  ---------------------+-------+------------+----------------------------------------------------
   Retrieve Request    |       |            | DHT Request for a value for a key
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'Q'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | PID   | 32 bytes   | Packet ID, used for delivery confirmation
                       | DTYP  | 1 byte     | Type of data to retrieve:
                       |       |            |   'I' = Index Packet
                       |       |            |   'E' = Email Packet
                       |       |            |   'K' = Email destination
                       | KEY   | 32 bytes   | DHT key to look up
  ---------------------+-------+------------+----------------------------------------------------
   Store Request       |       |            | DHT Store Request
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'S'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | PID   | 32 bytes   | Packet ID, used for delivery confirmation
                       | HLEN  | 2 bytes    | HashCash length
                       | HK    | byte[]     | HashCash token (HLEN bytes)
                       | DLEN  | 2 bytes    | Length of DATA field
                       | DATA  | byte[]     | Payload packet to store (DLEN bytes). Can be an
                       |       |            | Index Pkt / Email Pkt / Email Destination)
  ---------------------+-------+------------+----------------------------------------------------
   Find Close Peers    |       |            | Request for k peers close to a key
                       | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                       | TYPE  | 1 byte     | Value = 'F'
                       | VER   | 1 byte     | Format version (only ver. 1 is supported)
                       | PID   | 32 bytes   | Packet ID, used for delivery confirmation
                       | KEY   | 32 bytes   | DHT key
  ---------------------+-------+------------+----------------------------------------------------
```

### 3.2. Data Packets

These are always sent wrapped in a Communication Packet (see 3.1).
E denotes an encrypted field.

```
   Packet Type         | Field | Data Type  | Description
  ---------------------+-------+------------+----------------------------------------------------
   Email Packet        |       |            | An email or email fragment, 1 recipient.
   (encrypted)         | TYPE  | 1 byte     | Value = 'E'
                       | KEY   | 32 bytes   | The DHT key of the packet (generated randomly)
                       | DELP  | 32 bytes   | Deletion key in plaintext, zeroed out for retrieving
                       | LEN   | 2 bytes    | Length of the encrypted part of the packet
                       | DATA  | byte[]     | LEN bytes, encrypted with recipient's key
  ---------------------+-------+------------+----------------------------------------------------
   Email Packet        |       |            | Storage format for the Incomplete Email Folder.
   (unencrypted)       | TYPE  | 1 byte     | Value = 'U'
                       | DELP  | 32 bytes   | Deletion key in plaintext, zeroed out for retrieving
                       | DELV  | 32 bytes   | Deletion key for verification
                       | MSID  | 32 bytes   | Message ID in binary format
                       | FRID  | 2 bytes    | Fragment Index of this packet (0..NFR-1)
                       | NFR   | 2 bytes    | Number of fragments in the email
                       | MLEN  | 2 bytes    | Length of the MSG field
                       | MSG   | byte[]     | email content (MLEN bytes)
  ---------------------+-------+------------+----------------------------------------------------
   Index Packet        |       |            | Contains the DHT keys of one or more Email Packets.
                       | TYPE  | 1 byte     | Value = 'I'
                       | DH    | 32 bytes   | SHA-256 hash of the recipient's email destination
                       | NP    | 1 byte     | Number of keys in the packet
                       | KEY1  | 256 bytes  | DHT key of the first Email Packet
                       | KEY1  | 256 bytes  | DHT key of the second Email Packet
                       | ...   | ...        | ...
                       | KEYn  | 256 bytes  | DHT key of the n-th Email Packet
  ---------------------+-------+------------+----------------------------------------------------
   Relay Packet        |       |            | The payload of a Relay Request
                       | TYPE  | 1 byte     | Value = 'R'
                       | TMIN  | 4 bytes    | Earliest send time
                       | TMAX  | 4 bytes    | Latest sent time (no guarantee!)
                       | XK    | 32 bytes   | Key to XOR the payload with
                       | NEXT  | 384 bytes  | Destination to forward the packet to
                       | DLEN  | 2 bytes    | Length of DATA field in bytes, up to ~30k
                       | DATA  | byte[]   E | Payload, encrypted with NEXT's key (DLEN bytes)
                       |       |            | (can contain another Relay Packet, Email Packet,
                       |       |            | or Retrieve Request)
  ---------------------+-------+------------+-------------------------------------------------
   Peer List           |       |            | Response to a Find Close Peers Request or
                       |       |            | a Relay List Request
                       | TYPE  | 1 byte     | Value = 'L'
                       | NUMP  | 2 bytes    | Number of peers in the list
                       | P1    | 384 bytes  | Destination key
                       | P2    | 384 bytes  | Destination key
                       | ...   | ...        | ...
                       | Pn    | 384 bytes  | Destination key
  ---------------------+-------+------------+-------------------------------------------------
   Email Destination   |       |            | TBD
                       |       |            | 
                       |       |            | 
                       |       |            | 
                       |       |            | 
                       |       |            | 
  ---------------------+-------+------------+-------------------------------------------------
```

## 4. Protocols For Inter-Node Communication

### 4.1. Sending Email

I2P nodes involved: A=Sender, R1...Rn=Relays, S1...Sm=Storage Nodes

1. A sends an Relay Packet to R1.
2. R1 decrypts the packet, waits a random amount of time, and sends it to R2.
3. R2 confirms delivery with R1.
4. Repeat until packet arrives at Rn.
5. Rn decrypts the Relay Packet into an Email Packet.
6. Rn sends the packet to S1,...,Sm through a Kademlia STORE

### 4.2. Retrieving Email

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

Note: The system purposely does not prevent retrieval of email by unauthorized nodes, in order to
allow a node to test the reliability of a storage node before using it to store an Email Packet.

### 4.3. Pinging a node

### 4.4. Responding to a Ping

### 4.5. Storing an Email Packet on a storage node

### 4.6. Sending a Retrieve Request to a Node

### 4.7. Deleting an Email Packet

1. Recipient knows the deletion key after it decrypts the email packet (the packet also contains  
   a "plaintext deletion key" field, which is all zeros after it leaves a storage node).
2. Recipient sends a Delete Request to all storage nodes close to the DHT key of the Email Packet

### 4.8. Looking up an email address in the directory

### 4.9. Announcing a new email address to the directory

### 4.10. Updating or deleting an email address from the directory


## 5. Algorithms Used By Nodes Locally

### 5.1. Sending Email   *** OUTDATED ***
  
1. Given a MIME email, GZip it, sign it, store it in the local outbox, split it into 30kB  
   packets plus an Index Packet, pad the packets if necessary, and store them locally.
2. Pick a random packet/recipient combination for which the wait time is up, and for which  
   less than <storage redundancy> distinct delivery confirmations have been received within the  
   timeout period, and the maximum number of retries has not been reached.
3. Set a random packet ID on the packet.
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

1.  Randomly choose a set of n relay nodes,  R1...Rn (outgoing chain for request)
2.  Randomly choose a set of m relay nodes, S1...Sm.(return chain / inbound mail chain) - with  
    Sm being the node closest to the receiver and  
    S1 the chain node closest to fetcher
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
11. Set a random packet ID on the packet at each layer
12. Queue the packet for immediate sending to the first relay.
  
The following is done continuously and asynchronously, regardless of whether there a Retrieve
Request has been sent recently:
  
1. Nodes are always ready for incoming Email Packets.
2. For every email packet that is received, a Delete Request is sent out to the Kademlia network via a relay chain.
  
### 5.3. Fetching Email Packets For an Address Through Kademlia

1. Do a Kademlia lookup for the keypair in the Retrieve Request
2. Extract packet IDs from the Index Packet(s)
3. Do a Kademlia lookup for each packet ID
4. Check the packet ID on the Email Packets and send them back to the first return relay
  
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

* Wait 20 seconds for any emails to arrive, then return from HTTP request or POP3 request. All other emails will be received in the background.
* The "high priority" setting found in most email clients sets all relaying delays to zero.

## 9. Files

### 9.1. Identities file

Stores all email identities the local user has created. One line per entry.
Format: <base64 key><tab><public name>[<tab><description>][<tab><email address>]

## 10. To Do

* Replication per the Kademlia spec
* Have an option to disable automatic lookups in the distributed address directory for your own address
* Automatically create a different sender address for each recipient
* test relays periodically
* test storage nodes periodically  
* optionally pad all packets to a system-wide fixed size
* Always use the same amount of bandwidth, testing nodes or sending dummy messages if necessary.
* Always keep incoming bandwidth the same as outgoing bandwidth
* Pad packets before encrypting to prevent tagging attacks
* packet id for confirmation
* onion packets/regular packets
* delivery confirmation to sender
* What about incoming bandwith, can it be limited?
* Turnkey outproxy feature?
* All nodes keep track of I2PBote peers through Peer List Requests and by pinging known peers periodically.
* optionally, packets are padded to a constant size
* select different set of hops for each packet
* instead of sending dummy messages for cover traffic, test gateways by routing packet to self
* keep track of uptimes + reliability of paths (or gateways), have a "minimum uptime/min reliability percentage" setting for selecting gateways
* Use three different I2P destinations for sending and receiving data
* Use more storage nodes if email destination is known to be heavily frequented (like Kademlia does for popular hashes)
* mention that min/max delay = earliest/latest send time (explain that there is no guarantee for the send time being between min and max, it can be after max)
* Consider replacing ElGamal with 256-bit ECC encryption (=86 base64 chars) which comes standard in Java 7
* If an email fits in one packet, don't bother with an index packet
* Emails in the Outbox: Show addt'l "progress" progress column which is clickable and brings up detailed info.
* On first run, offer to create a new email identity, and optionally assign an email address to it.
* Folder structure on disk should reflect folders in app. All email folders visible to the user have the same root directory: [app dir]/folders.
* ask user for confirmation before deleting an email identity
* have a view of the incomplete folder, so users can look at incomplete emails
* use a key store for email identities so they are not locally readable?
