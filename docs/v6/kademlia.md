# Kademlia (DRAFT)

The Kademlia implementation used by I2P-Bote differs from standard Kademlia in several ways:

* Items can be deleted from the DHT
* No caching of DHT items because they are only retrieved once and then deleted
* I2P-Bote uses sibling lists (S-buckets) as suggested in the S/Kademlia paper

There are three types of data that is stored in the DHT:

* `Email Packet`
* `Index Packet`
* `Contact`

## Delete Verification

The `Delete Verification` code is the SHA-256 hash of another block of data called the `Delete Authorization` which is encrypted in the `Email Packet` and additionally used as a checksum for verification after decryption of `Email Packet`.

To delete an `Email Packet` or an entry in an `Index Packet` the delete request must contain the correct verification code (the `Delete Verification` hash).

This means no third party can delete the `Email Packet` or item in `Index Packet` until the recipient has decrypted the `Email Packet` and sent out a delete request containing the `Delete Authorization`. 

## Email Packet (encrypted)

`Email Packet` contain full (if size less than 30 KiB) or one part of MIME message with service information for processing, decrypting, and assembling email.
The DHT key of an `Email Packet` is the SHA-256 hash of the `LEN` and `DATA` fields of `Email Packet`.

To delete an `Email Packet` the delete request must contain the correct verification code (the `Delete Verification` hash).

Note that once a delete request for a given `Email Packet` has been received by a node that is storing the `Email Packet`, the node knows the `Delete Authorization` code and can propagate the delete request to other nodes that don't know yet that the `Email Packet` has been deleted.

## Index Packet

`Index Packets` contain the DHT keys of one or more `Email Packets`.  
The DHT key of an `Index Packet` is the SHA-256 hash of the `Email Destination` the `Email Packets` are destined for.

To check for new email for a given `Email Destination`, I2P-Bote first queries the DHT for `Index Packets` for that `Email Destination`, then queries the DHT for all `Email Packet` keys in the `Index Packets`.  
When a complete set of `Email Packets` has been received, the email is reconstructed and placed in the `inbox`.

To delete an item in `Index Packet` a `Delete Verification` hash is required as well.  
It is the same hash as the one for the `Email Packet` which the `Index Packet` entry points to.

## Contact

!! note "Note"

    For now only a part of Java I2P-Bote. WIP

DHT-propagated mapping for `public name` and `Email Destination`.
It is a decentralized analogue of the local address book.
