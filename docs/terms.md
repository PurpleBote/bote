# Glossary of Terms

## `Email Destination`

As the name implies, an `Email Destination` is an identifier by which somebody can be reached via I2P-Bote.

## `Email Address`

`Email Addresses` in I2P-Bote are shortcuts for `Email Destinations`.  
`Email Address`<-->`Email Destination` mappings are stored in two places:

- local address book;
- distributed address directory.

## `Email Identity`

An `Email Identity` is an `Email Destination` plus a name you want to be known as when you use that identity.   
Technically speaking, an `Email Identity` consists of four things:

* The two public keys of `Email Destination`;
* The two private keys of `Email Destination`;
* A public name which is shown to other people in emails;
* A description which is not shown to anybody but you (it helps you remember which `Email Identity` you use for which purpose).

An `Email Identity` is not required for sending emails (in that case only "Anonymous" can be set in the "sender" field).

## `I2P destination`

The address of a I2P-Bote node on the I2P network. There is normally no need to know it.  
Stored in file `destination.key` in working directory.  
`I2P destinations` and `Email Destinations` look similar, but are completely independent of each other.

## `Node`

A full-featured node that is capable of handling requests for storage, retrieval, deletion, etc.  
The "closest" to the DHT-keys is determined by the `I2P destination` of this nodes type.

## `Relay` (`Peer`)

A node with high availability in the near future (at least 18 hours in the last 24 hours).  
They are used as intermediaries for anonymized transmission of packets to `Nodes`.
