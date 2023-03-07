# 2. Kademlia

The Kademlia implementation used by I2P-Bote differs from standard Kademlia in several ways:

 * Items can be deleted from the DHT
 * No caching of DHT items because they are only retrieved once and then deleted
 * I2P-Bote uses sibling lists (s-buckets) as suggested in the S/Kademlia paper

There are three types of data that is stored in the DHT: Email Packets, Index Packets, and Contacts.

Index packets contain the DHT keys of one or more email packets. The DHT key of an index Packet is the SHA-256 hash of the Email Destination the email packets are destined for.   
To check for new email for a given Email Destination, I2P-Bote first queries the DHT for index packets for that Email Destination, then queries the DHT for all email Packet keys in the index packets.  
When a complete set of email packets has been received, the email is reconstructed and placed in the inbox.

TODO Contacts

To delete an email Packet, or an entry in an index Packet, the delete request must contain the correct verification code (the delete verification hash).   
The verification code is the SHA-256 hash of another block of data called the delete authorization which is encrypted in the email Packet.   
This means no third party can delete the Packet until the recipient has decrypted the Packet and sent out a delete request containing the delete authorization.   
Note that once a delete request for a given email Packet has been received by a node that is storing the email Packet, the node knows the delete authorization code and can propagate the delete request to other nodes that don't know yet that the Packet has been deleted.

To delete an index Packet entry, a delete verification hash is required as well. It is the same hash as the one for the email Packet which the index Packet entry points to.
