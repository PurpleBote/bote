# 6. Files

## 6.1. Identities file

Stores all email identities the local user has created. The file uses the Java Properties format, and can be read into / written out from a Java Properties object.

The following property keys are currently stored and recognized:

* identity#.publicName  - The public name of the identity, included in emails.
* identity#.key         - Base64 of the identity keys.
* identity#.salt        - Salt used to generate a fingerprint.
* identity#.description - Description of the identity, only displayed locally.
* identity#.picture     - Base64 of byte[] containing picture data
* identity#.text        - Text associated with the identity.
* identity#.published   - Has the identity been published to the public addressbook?
  (# is an integer index)

The base64 key contains two public keys (encryption+signing) and two private keys.

The identities file can optionally contain the property key "default", with its value set to an Email Destination (i.e. two public keys).   
If the Email Destination matches one of the identities, that identity is used as the default.

ToDo: Looks specific to Java app, remove from protocol description 

## 6.2. Included .jar files and third-party source code

```
bcprov-jdk15on-152.jar     The BouncyCastle library and SecurityProvider.
mailapi-1.5.4.jar          Part of JavaMail 1.5.4
lzma-9.20.jar              An LZMA implementation from http://www.7-zip.org/sdk.html
ntruenc-1.2.jar            An NTRU implementation from http://sf.net/projects/ntru/
                           (only the NTRUEncrypt part, not NTRUSign)
flexi-gmss-1.7p1.jar       A stripped down and patched version of the FlexiProvider
                           sources (http://www.flexiprovider.de/), containing only the
                           classes needed for GMSS.
                           The patch consists of a modified de.flexiprovider.core.CoreRegistry
                           and de.flexiprovider.api.Registry and its purpose is to
                           reduce dependencies on classes not related to GMSS.
scrypt-1.4.0.jar           An scrypt implementation from https://github.com/wg/scrypt/
```
