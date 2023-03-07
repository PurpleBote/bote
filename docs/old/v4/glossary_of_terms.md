# 9. Glossary of Terms

## Email Destination

As the name implies, an Email Destination is an identifier by which somebody can be reached via I2P-Bote.   
An Email Destination is a Base64 string containing a public encryption key and a signature verification key.   
Example of a 512-character Email Destination (ElGamal-2048/DSA-1024):
  
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

Example of a 86-character Email Destination (ECC-256):

```
  1Lcvly8no5of6juJKxqy-xA-MStM2c2XKorepH1oqs5yKBkg9-ZcG4G4kZY1E~2672cMA806l9EicQLmlehB1m
```

## Email Address

Email Addresses in I2P-Bote are shortcuts for Email Destinations.   
Email Address <--> Email Destination mappings are stored in two places: the local address book and the distributed address directory.

## Email Identity
An Email Identity is an Email Destination plus a name you want to be known as when you use that identity.   
Technically speaking, an Email Identity consists of four things:

* An Email Destination (i.e. two public keys)
* The two private keys for the Email Destination
* A public name which is shown to other people in emails
* A description which is not shown to anybody but you.
  It helps you remember which Email Identity you use for which purpose.

An email identity is not required for sending emails (in that case only "Anonymous" can be selected in the "sender" field).

## I2P destination

The address of a I2P-Bote node on the I2P network. There is normally no need to know it.   
I2P destinations and Email Destinations look similar, but are completely independent of each other.
