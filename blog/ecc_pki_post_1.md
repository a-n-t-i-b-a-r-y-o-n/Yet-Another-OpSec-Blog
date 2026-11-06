# Intro: Asymmetric Cryptography in Security Operations

Webster’s Dictionary defines… *just kidding*!

At the core, “security operations” is the practice of establishing and defending *trust* inside an organization. When an analyst reviews a SIEM alert, a packet capture, a software update, or a stream of endpoint telemetry, the underlying question is almost always the same:

> “Can I trust that this thing is what it claims to be, that it came from where it claims to have come from, and that it has not been altered in transit?”

That line of questioning has a precise technical answer, which can be assured by cryptography. One common form of cryptography, asymmetric (i.e. “public-key”) cryptography, is used heavily within the tech industry. It leverages dedicated infrastructure to issue, distribute, and revoke its keys: the Public Key Infrastructure, aka “PKI”.

## The issue with symmetric cryptography

Asymmetric cryptography stands in contrast to *symmetric* cryptography, where both the sender and receiver use a single shared secret for both encryption and decryption.

On the one hand, it is usually fast, well-understood, and a workhorse of bulk data protection. On the other, it has a fundamental weakness: **the key distribution problem**.

If Alice and Bob want to exchange encrypted data, they must somehow agree on a shared secret *before* they can communicate securely. In a threat model where the channel between them is untrusted (and in security operations it always is, by assumption) there is no obvious way to do that exchange safely. This is a bootstrap problem – you need a secure channel to establish your secure channel.

Individuals and large organization runs into this constantly, worldwide. An endpoint in one data center needs to talk to a service in a different region. A new contractor's laptop needs to authenticate to internal systems. A deployment pipeline needs to verify that a build artifact really came from the build server and not an attacker who gained a foothold in CI. A journalist needs to receive encrypted messages from anonymous sources. Doing this by hand - emailing keys, pasting secrets into tickets or on websites, hardcoding shared tokens - is exactly the kind of fragile, unauditable process that security operations exists to eliminate.

## Asymmetric cryptography: disparate keys

Asymmetric cryptography replaces the single shared secret with a pair of mathematically linked keys:

- A **private** key that the owner keeps secret.
- A **public** key that anyone can have and read.

The critical property is that any data encrypted (or signed) with one member of the pair can only be decrypted (or validated) by the other.

This single concept solves two distinct problems at once:

1. **Confidentiality.** Anyone can encrypt a message using Alice's *public* key, but only Alice's *private* key can decrypt it. This means Alice and Bob can establish a secure channel over a hostile network without ever having shared a secret in advance.
2. **Authentication and integrity (digital signatures).** If Alice signs a message with her *private* key, anyone can verify the signature using her *public* key. A valid signature proves two things: the message was produced by whoever holds the private key, and it has not been modified since it was signed. This is the foundation of non-repudiation.

## How this forms the bedrock

Like I mentioned, these two capabilities are not purely academic. They map directly onto what an operational security system or practitioner must do every day:

- **Identity.** PKI is how a system, a user, or a service cryptographically proves *who it is*. TLS server certificates, code-signing certificates, and mutually authenticated (mTLS) service identities all rest on key pairs. Without a trusted way to bind a public key to an identity, "authentication" degrades into something much weaker.
- **Integrity of artifacts.** Build pipelines, configuration management, and incident forensics all rely on signatures to prove that a binary, config file, or log extract is unaltered. When an investigator pulls an image or a log bundle, a valid signature is what separates "this is the real evidence" from "this could have been swapped."
- **Confidentiality at scale.** Transport Layer Security (TLS) - the protocol that secures web traffic, API calls, and much internal service-to-service communication - is built on top of public-key cryptography for the handshake and key agreement. Secure internal traffic is a prerequisite for detecting anomalies in the first place, because encrypted, authenticated channels are what make the data trustworthy.
- **Non-repudiation and audit.** Signed records support accountability. In a post-incident review, a signed timestamp or signed log entry is far stronger evidence than an unsigned one, because it resists both tampering and later denial.

## The remaining question PKI answers

There is one more problem that a raw key pair does not solve by itself:

> How do you know that a public key actually belongs to the entity it claims to?

Anyone can generate a key pair. An attacker could generate their own "Alice" key and try to pass it off as the real one.

Public key infrastructure answers this by having a trusted third party - a “Certificate Authority” (CA) - bind a public key to an identity inside a digital certificate, and sign that binding. Because you trust the CA, you can trust the identity the certificate asserts. The CA hierarchy, the certificate format, the issuance and revocation process, and the validation rules together form the PKI. Modern browsers ship with a “trust store” of trusted CAs.

## Summary

As you can see, PKI is not some niche cryptography topic for arcane, oblique security nerds. It’s actually vital for security operations. It is the mechanism by which the entire modern trust model - secure web traffic, verified software, authenticated services, signed evidence - actually functions. Understanding how PKI is built, and how it can fail, is a prerequisite for operating it securely, for detecting when it is being abused, and for responding when it breaks.

Since this concept is around us at all time, securing all manner of operations, it seemed like a good topic to discuss in my blog. This post is the first of many that will explain PKI and encryption w.r.t. security operations. I hope to publish at least one a week!

Stay tuned for more!


## Further reading

Here's some good jump-off points for further reading on this stuff:
* [Wikipedia: Public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography)
* [Wikipedia: Symmetric-key algorithm](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)
* [Wikipedia: Diffie-Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
* [Wikipedia: One-way function](https://en.wikipedia.org/wiki/One-way_function)