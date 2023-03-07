# Cryptography (DRAFT)

## 1. Asymmetric encryption and signing

Encryption and signature algorithms:

| ID | Crypto           | Signing     | Symmetric | Hash    | Java Bote | pboted             |
|----|------------------|-------------|-----------|---------|-----------|--------------------|
| 1  | ElGamal-2048     | DSA-1024    | AES-256   | SHA-256 | active    | never (deprecated) |
| 2  | ECDH-256         | ECDSA-256   | AES-256   | SHA-256 | active    | active             |
| 3  | ECDH-521         | ECDSA-521   | AES-256   | SHA-512 | active    | active             |
| 4  | NTRUEncrypt-1087 | GMSS-512    | AES-256   | SHA-512 | active    | soon               |
| 5  | X25519           | ED25519     | AES-256   | SHA-512 | no        | active             |

## 2. Email Identity Formats

After the introduction of new, more modern algorithms, the question arose of the ambiguity of determining the type of `Email Destination` by its length in the form of Base64.  

A more comprehensive format has been developed that will later allow combinations of different types of keys, not just predefined ones.

The previously used option for storing and passing `Email Destinations` and `Email Identities` is now version 0.

### Version 0

Used by **pboted** and **Java Bote**

- An `Email Destination` is a Base64 string containing:
    - public encryption key;
    - public signature verification key.
- An `Email Identity` is a Base64 string containing:
    - public encryption key;
    - public signature verification key;
    - private encryption key;
    - private signature key.

Example of a 512-character `Email Destination` (ElGamal-2048/DSA-1024):
  
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

Example of a 86-character `Email Destination` (ECC-256):

```
  1Lcvly8no5of6juJKxqy-xA-MStM2c2XKorepH1oqs5  
  yKBkg9-ZcG4G4kZY1E~2672cMA806l9EicQLmlehB1m
```

`Email Destination` type can only be determined by the length of the base64 string:

| ID | Public Part | Private Part |
|----|-------------|--------------|
| 1  | 512         | 880          |
| 2  | 86          | 172          |
| 3  | 174         | 348          |
| 4  | 2079        | 97813        |

### Version 1

Used by **pboted**

Template:
`<data format>.<encoded data>`

- data format - Can be (for now):
    - **b32** (`Base32`)
    - **b64** (`Base64`)
- encoded data - Can be bytes with:
    - `Email Destination` (*public keys only*)
    - `Email Identity` (*public and private keys*)

#### `Email Destination` format

| Field    | Size   | Description                                           |
|----------|--------|-------------------------------------------------------|
| `VER`    | 1 byte | format version                                        |
| `CTYPE`  | 1 byte | cryptography algorithm type                           |
| `STYPE`  | 1 byte | signing algorithm type                                |
| `SMTYPE` | 1 byte | symmetric encryption algorithm type                   |
| `HTYPE`  | 1 byte | hash algorithm type                                   |
| `CDATA`  | N byte | crypto public key (field length depends on the type)  |
| `SDATA`  | M byte | signing public key (field length depends on the type) |

Example:

```
b64.AQUFAgJI2e47rS9VkSEfM4ZHfuimMA1EoUmLZd1DNSX8fqocLA3v8cv~XSnQpUSax~9Gs2cFH2rtNOZekhF4i2RQ7QOI
```

#### `Email Identity` format

| Field    | Size   | Description                                           |
|----------|--------|-------------------------------------------------------|
| `VER`    | 1 byte | format version                                        |
| `CTYPE`  | 1 byte | cryptography algorithm type                           |
| `STYPE`  | 1 byte | signing algorithm type                                |
| `SMTYPE` | 1 byte | symmetric encryption algorithm type                   |
| `HTYPE`  | 1 byte | hash algorithm type                                   |
| `CDATA`  | N byte | crypto public key (field length depends on the type)  |
| `SDATA`  | M byte | signing public key (field length depends on the type) |
| `CPDATA` | X byte | crypto private key (field length depends on the type) |
| `SPDATA` | Y byte | signing private key (field length depends on the type)|

Example:

```
b64.AQUFAgJI2e47rS9VkSEfM4ZHfuimMA1EoUmLZd1DNSX8fqocLA3v8cv~XSnQpUSax~9Gs2cFH2rtNOZekhF4i2RQ7QOISPTr4NDUvlFrAt3SfPtZe6iF-NvfKlkuB8caWNqhR1jvOTDon2w~pDh~0yloEMkwONyugyKmTgscruV4SaAfBQ==
```

## 2. Fingerprints For Directory Entries

ToDo: Looks specific to Java app, remove from protocol description

TODO
H = scrypt(name, dest, zuf.wert); die letzten 8 Binärstellen von H müssen 0 sein
13*7+22+18 = 131
