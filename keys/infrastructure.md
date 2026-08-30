# Key Infrastructure

## Algorithms

Symmetric Key:  AES-256 (Required by ML-KEM-768)  
Asymmetric Key: ML-KEM-768  
Signatures:     FN-DSA-512 (Falcon-512)  
Fingerprints:   SHA-1 (160-bit / 20 byte)

## Trust

With the decentralized nature of Tapistri, chains of trust are formed through transparency and having a longstanding
and reputable account on multiple instances, with instances attesting to an entity's identity. The trust of an entity can
be visualized as a hub and spoke, where the hub is the entities Identity Key, being attested to by Instance Attestation 
ertificates and Historical Key Records.

### [Identity Key](./recovery.md)

The identity key establishes the root of trust for an user, and is only used in one of the following cases:
- Client is starting a new chain of HKRs on an instance
- Client is requesting an instance to revoke a set of keys
- Client needs to indisputably attest their identity

This key should be stored somewhere safe, preferably on a hardware key (Yubico YubiKey®, smartcard, etc) or offline.
There is no mechanism to recover or change this key without creating a new identity.

### Proxy Keys 

The proxy keys are the general-use keys that are used for message encryption. The entities have one signing
identity key that is rotated out, while having many encrypting keys on a per room basis.

### [Historical Key Records (HKRs)](./rotation.md)

Historical Key Records consist of three components:
- The new public key that they are introducing
- The signature of the old key that they are replacing (or the signature of the identity key)
- The validity period of the key (the time the key activates and expires)

These form a chain of keys that prove that the current key has a connection to the original key used on the account.

### Instance Attestation Certificates (IACs)

Instances have a public and private key used to attest to user key modifications, and to attest to the authenticity of a user. These records contain the following information:
- The signature of the instance
- The signature of the administrator or member authorizing the attestation
- The signature of the user being attested
- The time at which the attestion was signed

TODO: key rotation