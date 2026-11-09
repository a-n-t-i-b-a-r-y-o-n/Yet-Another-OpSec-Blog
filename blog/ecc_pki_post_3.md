# Intro: Digital Signatures

So far, we’ve seen why asymmetric cryptography matters w.r.t. security operations. In the last post, we looked at how to generate asymmetric key pairs and use them for encryption. In this post, we’ll look at how digital signatures work to ensure the integrity of received encrypted messages

Like I said in the previous post, it’s surprisingly common for folks to mix up encryption and signing in practice. Not the *concepts themselves* per se, but people definitely mix up the way in which the keys are used.

The amount of times I’ve worked with experienced engineers who claim their SSH key isn’t working because they uploaded their private key somewhere and are trying to authenticate with their public key 🙄

## Signing for authentication and integrity

\> A quick aside for any nontechnicals in the crowd: “Authorization” is what someone/something is or isn’t allowed to do. “Authentication” is proving that they are who they say they are. “Authentification” is a nonsense word made up by project managers.

So, signing uses the key pair in the diametrically opposite direction from encryption, which is where people end up getting confused:

- Sign with the sender's private key.
- Verify with the sender's public key.

Note that a digital signature is *not* an encryption of the message. It is a short value computed over a message (technically, over the hash of a message) using the sender's private key. This allows anyone with the sender's public key to check the signature. If verification succeeds, two things are established:

1. **Authentication**

The signature could only have been produced by someone holding the matching private key

1. **Integrity**

If even a single solitary bit of the message changes after signing, verification would not succeed.

Obviously, signing and encrypting are independent operations.

You can sign without encrypting to yield a public, tamper-evident attestation, encrypt without signing for a confidential message with no cryptographic proof of origin, or do both. In security operations, the signing direction is often the more important one, because so much of the job is about proving that data has not been tampered with and that it came from a trusted source. Think: signed build artifacts, signed log bundles, signed configuration, signed evidence, or anything else that you’d like to check the provenance of.

## Hash-then-sign

You may’ve caught it a second ago, but signatures are typically computed over a *hash* of the message, not the whole message itself. The signer hashes the data with a cryptographic hash function (e.g. SHA-256 and friends) and signs the resulting digest. The verifier recomputes the hash and checks the signature over it.

Two benefits here. One, it makes signing a lot more efficient - you sign a short fixed-size digest rather than a large file. Two, it binds the signature to the exact content via the hash's avalanche property: a tiny change in the input produces an entirely different hash, so any tampering is detected.

In case it isn’t obvious though, this does hinge the strength of the signing on the strength of the hash function. Be sure to avoid outdated, broken hash functions like SHA-1 or RC3.

My personal preference is for elliptic curve cryptography, which uses much shorter key lengths (and so has faster run times) than standard RSA. As an added upshot, Ed25519 (an EdDSA signature) does not require you to pre-hash the message the way classic RSA or ECDSA do. Ed25519's construction hashes internally as part of the signature scheme, so with an Ed25519 key you can sign full, raw plaintext data directly. So, on top of being faster, this prevents any mistakes w.r.t. hashing incorrectly or worse. Imagine trying to debug your cryptofunctions only to find that you were hashing with a newline on one end and without it on the other 😬

## Use Cases

This handy table breaks down the use cases you’ll face:

| Goal | I’m the \_\_\_ | Use | Result |
| --- | --- | --- | --- |
| Keep a secret from everyone but Bob | Sender | Encrypt with Bob's **public** key | Only Bob's **private** key decrypts |
| Prove I am me and the data is intact | Signer | Sign with my **private** key | Anyone verifies with my **public** key |

Hopefully you, like me, see the elegance of this simple symmetry. The same two keys, used in two opposite directions, give you both confidentiality and trust. Every certificate, every TLS session, every signed artifact in an organization is built from these two operations.

## Summary

We’ve now looked at the 2 primary use cases for asymmetric cryptography in security operations: encryption/decryption, and authentication/signing. These two applications underpin (or at least play a significant role in) the overwhelming majority of cryptographic operations today.

In the next post, we’ll do an *even deeper* dive on a modern family of algorithms: elliptic curves! If you like math, stay tuned!

See you in the next post!

## About the author

Hi there! 👋

My name is Braden Wright, and I'm software and security engineer with 5+ years of security engineering experience and more than double that in software. I spend my work days building security infrastructure and tooling, spotting ne'erdowells in SIEM logs, and injecting security into the software development process. I spend my days off writing software and firmware, prototyping hardware boards plus 3D modelling for their enclosures, and trying to "touch grass" in local hiking trails or state parks.

Despite the advent of generative AI, I still enjoy writing longform text and software by hand, key-by-key. So, I'm starting this little blog to document things I've learned in the industry. Hope you enjoy!
