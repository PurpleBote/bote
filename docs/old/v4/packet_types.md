# 3. Packet Types

As of release 0.2.5, the protocol version is 4. It is incompatible to earlier versions.   
Version 4 clients will talk to higher versions. There will be at least one more protocol version at some point in the future; this version will be backwards compatible to version 4.   
All packets start with one byte for the Packet type_, followed by one byte for the Packet version.   
Strings in packets are UTF8 encoded.   

## 3.1. Data Packets

These are always sent wrapped in a Communication Packet (see 3.2).   
E denotes encrypted data.   
For supported encryption and signature algorithms, see 7.1   

```
   Packet Type        | Field | Data Type  | Description
  --------------------+-------+------------+--------------------------------------------------------
   Email Packet,      |       |            | An email or email fragment, one recipient. Part of
   encrypted          |       |            | the Packet is encrypted with the recipient's key.
                      | TYPE  | 1 byte     | Value = 'E'
                      | VER   | 1 byte     | Protocol version
                      | KEY   | 32 bytes   | DHT key of the Packet, SHA-256 hash of LEN+DATA
                      | TIM   | 4 bytes    | The time the Packet was stored on a storage node
                      | DV    | 32 bytes   | Delete verification hash, SHA-256 hash of DA
                      | ALG   | 1 byte     | ID number of the encryption algorithm used
                      | LEN   | 2 bytes    | Length of DATA
                      | DATA  | byte[]   E | Data for decryption and Unencrypted Email Packet
                      |       |            | ToDo: add info about alg-specific prefixes
  --------------------+-------+------------+--------------------------------------------------------
   Email Packet,      |       |            | Storage format for the Incomplete Email Folder
   unencrypted        |       |            | 
                      | TYPE  | 1 byte     | Value = 'U'
                      | VER   | 1 byte     | Protocol version
                      | MSID  | 32 bytes   | Message ID in binary format
                      | DA    | 32 bytes   | Delete authorization key, randomly generated
                      | FRID  | 2 bytes    | Fragment Index of this Packet (0..NFR-1)
                      | NFR   | 2 bytes    | Number of fragments in the email
                      | MLEN  | 2 bytes    | Length of the MSG field
                      | MSG   | byte[]     | email content (MLEN bytes)
  --------------------+-------+------------+--------------------------------------------------------
   Index Packet       |       |            | Contains the DHT keys of one or more Email Packets,
                      |       |            | a Delete Verification Hash, and time stamp for each
                      |       |            | DHT key.
                      | TYPE  | 1 byte     | Value = 'I'
                      | VER   | 1 byte     | Protocol version
                      | DH    | 32 bytes   | SHA-256 hash of the recipient's email destination
                      | NP    | 4 bytes    | Number of entries in the Packet
                      | KEY1  | 32 bytes   | DHT key of the first Email Packet
                      | DV1   | 32 bytes   | Delete verification hash for KEY1
                      |       |            | (SHA-256 hash of DA in the email Packet)
                      | TIM1  | 4 bytes    | The time the key KEY1 was added
                      | KEY2  | 32 bytes   | DHT key of the second Email Packet
                      | DV2   | 32 bytes   | Delete verification hash for KEY2
                      |       |            | (SHA-256 hash of DA in the email Packet)
                      | TIM2  | 4 bytes    | The time the key KEY2 was added
                      | ...   | ...        | ...
                      | KEYn  | 32 bytes   | DHT key of the n-th Email Packet
                      | DVn   | 32 bytes   | Delete verification hash for KEYn
                      |       |            | (SHA-256 hash of DA in the email Packet)
                      | TIMn  | 4 bytes    | The time the key KEYn was added
  --------------------+-------+------------+--------------------------------------------------------
   Deletion Info      |       |            | Contains information about deleted  DHT items, which
   Packet             |       |            | can be Email Packets or Index Packet entries.
                      | TYPE  | 1 byte     | Value = 'T'
                      | VER   | 1 byte     | Protocol version
                      | NP    | 4 bytes    | Number of entries in the Packet
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
  --------------------+-------+------------+--------------------------------------------------------
   Peer List          |       |            | Response to a Find Close Peers Request or a
                      |       |            | Peer List Request
                      | TYPE  | 1 byte     | Value = 'L'
                      | VER   | 1 byte     | Protocol version
                      | NUMP  | 2 bytes    | Number of peers in the list
                      | P1    | 384 bytes  | Destination key
                      | P2    | 384 bytes  | Destination key
                      | ...   | ...        | ...
                      | Pn    | 384 bytes  | Destination key
  --------------------+-------+------------+--------------------------------------------------------
   Directory Entry    |       |            | A name/Email Destination mapping
   (Contact.java)     | TYPE  | 1 byte     | Value = 'C'
                      | VER   | 1 byte     | Protocol version
                      | KEY   | 32 bytes   | DHT key (SHA-256 hash of the UTF8-encoded name in
                      |       |            | lower case)
                      | DLEN  | 2 bytes    | Length of the DEST field
                      | DEST  | byte[]     | Email destination, 8 bit encoded (not base64)
                      | SALT  | 4 bytes    | A value such that scrypt(KEY|DEST|SALT) ends with
                      |       |            | a zero byte
                      | PLEN  | 2 bytes    | Length of PIC, max 7500
                      | PIC   | byte[]     | User picture in a browser-compatible format
                      |       |            | (JPG, PNG, GIF)
                      | COMP  | 1 byte     | Compression type_ for TEXT
                      | TLEN  | 2 bytes    | Length of the TEXT field, max 2000
                      | TEXT  | byte[]     | User defined text in UTF8
  --------------------+-------+------------+--------------------------------------------------------
```

## 3.2. Communication Packets

Communication packets are used for sending data between two I2P-Bote nodes.   
They contain a data Packet, see 3.1.   
All Communication Packets start with a four-byte prefix, followed by one byte for the Packet type, one byte for the Packet version, and a 32-byte Correlation ID.   

```  
   Packet Type        | Field | Data Type  | Description
  --------------------+-------+------------+--------------------------------------------------------
   Relay Request      |       |            | A Packet that tells the receiver to communicate with
                      |       |            | a peer, or peers, on behalf of the sender. It contains
                      |       |            | an I2P destination, a delay time, and a Communication
                      |       |            | Packet.
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'R'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for responses
                      | HLEN  | 2 bytes    | HashCash length
                      | HK    | byte[]     | HashCash token (HLEN bytes)
                      | DLY   | 4 bytes    | Delay in seconds before sending
                      | NEXT  | 384 bytes  | Destination to forward the Packet to
                      | RET   | Ret. Chain | An empty or non-empty Return Chain
                      | DLEN  | 2 bytes    | Length of the DATA field in bytes
                      | DATA  | byte[]   E | Encrypted with the recipient's public key. Can contain
                      |       |            | another Relay Request, a Store Request, or a Deletion
                      |       |            | Query.
                      | PAD   | byte[]     | Optional padding
  --------------------+-------+------------+--------------------------------------------------------
   Relay Return       |       |            | Contains a Return Chain and a payload_. The final
   Request            |       |            | destination is unknown to the sender or any of the
   (not implemented)  |       |            | hops. Used for responding to a relayed data Packet.
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'K'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for confirmation between two
                      |       |            | relay peers. A new correlation ID is set at every hop.
                      | RET   | Ret. Chain | A non-empty Return Chain
                      | DLEN  | 2 bytes    | Length of the DATA field in bytes
                      | DATA  | byte[]   E | The payload. AES256-encrypted at every hop in the
                      |       |            | return chain.
  --------------------+-------+------------+--------------------------------------------------------
   Return Chain       |       |            | Not a Packet itself, but part of a Relay (Return)
   (not implemented)  |       |            | Request
                      | RL1   | 2 bytes    | Length of RET1. Zero means an empty return chain,
                      |       |            | nothing follows.
                      | RET1  | RL1 bytes  | Hop 1 in the return chain. Contains a zero byte,
                      |       |            | followed by the hop's I2P destination and an AES key
                      |       |            | to encrypt the payload_ with.
                      | RL2   | 2 bytes   E| Length of RET2.
                      | RET2  | RL2 bytes E| Hop 2 in the return chain. Contains a zero byte,
                      |       |            | followed by the hop's I2P destination and an AES key
                      |       |            | to encrypt the payload_ with. This field is encrypted
                      |       |            | with the PK of hop 2.
                      | RL3   | 2 bytes   E| Length of RET3.
                      | RET3  | RL3 bytes E| Hop 3 in the return chain. Contains a zero byte,
                      |       |            | followed by the hop's I2P destination and an AES key
                      |       |            | to encrypt the payload_ with. This field is encrypted
                      |       |            | with the PKs of hops 3 and 2.
                      | ...   | ...        | ...
                      | RLn   | 2 bytes   E| Length of RETn.
                      | RETn  | RLn bytes E| Last hop in the return chain. Contains a zero byte,
                      |       |            | followed by the hop's I2P destination and an AES key
                      |       |            | to encrypt the payload_ with. This field is encrypted
                      |       |            | with the PKs of hops n...2.
                      | RLn+1 | 2 bytes   E| Length of RETn+1.
                      | RETn+1| RLn+1 byt E| Number of hops (>0) followed by all AES256 keys
                      |       |            | (one per hop)
                      | RLn+2 | 2 bytes    | Value=0 to indicate this is the end of the return chain
  --------------------+-------+------------+--------------------------------------------------------
   Fetch Request      |       |            | Request to a chain endpoint to retrieve an Index 
   (not implemented)  |       |            | Packet or Email Packet.
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = ''
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for responses
                      | DTYP  | 1 byte     | Type of data to retrieve
                      | KEY   | 32 bytes   | DHT key to look up
                      | KPR   | 384 bytes  | Email keypair of recipient
                      | RLEN  | 2 bytes    | Length of the RET field
                      | RET   | byte[]     | Relay Packet, contains the return chain (RLEN bytes)
  --------------------+-------+------------+--------------------------------------------------------
   Response Packet    |       |            | Response to a Retrieve Request, Fetch Request,
                      |       |            | Find Close Peers Request, or a Peer List Request
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'N'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID of the request Packet
                      | STA   | 1 byte     | Status code:
                      |       |            |   0 = OK
                      |       |            |   1 = General error
                      |       |            |   2 = No data found
                      |       |            |   3 = Invalid Packet
                      |       |            |   4 = Invalid HashCash
                      |       |            |   5 = Not enough HashCash provided
                      |       |            |   6 = No disk space left
                      | DLEN  | 2 bytes    | Length of the DATA field; can be 0 if no payload
                      | DATA  | byte[]     | A Data Packet
  --------------------+-------+------------+--------------------------------------------------------
   Peer List Request  |       |            | A request for a list of
                      |       |            | high-reachability relay peers
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'A'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for responses
  --------------------+-------+------------+--------------------------------------------------------
```

###  DHT Communication Packets
  
```
   Packet Type        | Field | Data Type  | Description
  --------------------+-------+------------+--------------------------------------------------------
   Retrieve Request   |       |            | A request to a peer to return a data item for a given
                      |       |            | DHT key and data type
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'Q'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for responses
                      | DTYP  | 1 byte     | Type of data to retrieve:
                      |       |            |   'I' = Index Packet
                      |       |            |   'E' = Email Packet
                      |       |            |   'C' = Directory Entry
                      | KEY   | 32 bytes   | DHT key to look up
  --------------------+-------+------------+--------------------------------------------------------
   Deletion Query     |       |            | A request to a peer to return a Deletion Info Packet 
                      |       |            | for a given Email Packet DHT key. If the Email Packet 
                      |       |            | is not known to be deleted, no response is expected
                      |       |            | back.
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'Y'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for responses
                      | KEY   | 32 bytes   | DHT key to look up
  --------------------+-------+------------+--------------------------------------------------------
   Store Request      |       |            | DHT Store Request
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'S'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for responses
                      | HLEN  | 2 bytes    | HashCash length
                      | HK    | byte[]     | HashCash token (HLEN bytes)
                      | DLEN  | 2 bytes    | Length of the DATA field
                      | DATA  | byte[]     | Data Packet to store (DLEN bytes).
                      |       |            | Can be an Index Packet, Email Packet, or Email
                      |       |            | Destination.
  --------------------+-------+------------+--------------------------------------------------------
   Email Packet       |       |            | Request to delete an Email Packet by DHT key
   Delete Request     |       |            |
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'D'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for responses
                      | KEY   | 32 bytes   | DHT key of the Email Packet to delete
                      | DA    | 32 bytes   | Delete Authorization (SHA-256 must equal DV in the
                      |       |            | email pkt)
  --------------------+-------+------------+--------------------------------------------------------
   Index Packet       |       |            | Request to remove one or more entries (Email Packet 
   Delete Request     |       |            | keys) from an Index Packet
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'X'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for responses
                      | DH    | 32 bytes   | The Email Destination hash of the Index Packet
                      | N     | 1 byte     | Number of entries in the Packet
                      | DHT1  | 32 bytes   | First DHT key to remove
                      | DA1   | 32 bytes   | Delete Authorization (SHA-256 must equal DV in the 
                      |       |            | email Packet referenced by DHT1)
                      | DHT2  | 32 bytes   | Second DHT key to remove
                      | DA2   | 32 bytes   | Delete Authorization (SHA-256 must equal DV in the 
                      |       |            | email Packet referenced by DHT2)
                      | ...   | ...        | ...
                      | DHTn  | 32 bytes   | n-th DHT key to remove
                      | DAn   | 32 bytes   | Delete Authorization (SHA-256 must equal DV in the 
                      |       |            | email Packet referenced by DHTn)
  --------------------+-------+------------+--------------------------------------------------------
   Find Close Peers   |       |            | Request for k peers close to a key
                      | PFX   | 4 bytes    | Packet prefix, must be 0x6D 0x30 0x52 0xE9
                      | TYPE  | 1 byte     | Value = 'F'
                      | VER   | 1 byte     | Protocol version
                      | CID   | 32 bytes   | Correlation ID, used for responses
                      | KEY   | 32 bytes   | DHT key
  --------------------+-------+------------+--------------------------------------------------------
```
