# Intro: Key Pairs and Encryption

The previous post established why exactly asymmetric cryptography matters to security operations. In this post, we’ll begin to dive deeper into the mechanics. We’ll look at how a key pair is created, how encryption actually works in practice. There’s actually two main uses of asymmetric keypairs – encryption/decryption and signing – so this post starts by discussing the former.

We’re still at the conceptual level here so, while we won’t get into technical details and OpenSSL commandline invocations today, rest assured that they’re coming.

NB: Mixing up encryption and signing is a surprisingly common mistakes in the field, which can have grave operational consequences. Despite being nearly opposite uses of the keypair, they may seem similar to the uninitiated, so I’m not even going to put them in the same post! 👉😏

## Key generation

Let’s start by talking about how to even *make* one of these key pairs we’ve been talking about.

A key pair is created by a key-generation algorithm like RSA or Ed25519. The algorithm picks a random private key and then deterministically derives the matching public key from it with a standardized mathematical relationship.

Two big things come into play:

- **The private key must be generated from a cryptographically-hardened, secure source of randomness**

Not all random number generators (RNGs) are considered cryptographically secure. If the entropy source is weak, the "random" private key may be guessable, and the entire source of protection collapses on itself. This is not theoretical at all! Historical failures of key generation (predictable seeds, reused nonces, insufficient or no “salt”) have led to real compromises.

- **Deriving the private key from the public key must be computationally infeasible.**

That one-way relationship is the whole basis of the security scheme. The difficulty comes from a sufficiently hard “trapdoor problem” that the chosen algorithm is built on, e.g. integer factorization for RSA, the discrete logarithm (DLP) problem for classical DSA/ElGamal, and the elliptic-curve discrete logarithm problem (ECDLP) for ECC. These problems are easily solved if you know the answer, but deriving the answer with no hints is intended to be computationally infeasible on modern hardware.

In modern cryptosystems, the details of the algorithm are public knowledge. That is, we’re not relying on any form of secrecy about the encryption process itself, just the mathematical relationship between the keys.

For an organization’s operations team, the practical takeaway is the understanding that a lot of PKI risk originates in the key generation process. Where keys are generated, how the randomness is sourced, how the private key is stored and protected, and who has access to the generation host (and e.g. its command history) all matter as much as the algorithm choice. The exact algorithm used should be considered public knowledge.

## Asymmetric encryption

Once we have generated a key pair, the two keys are used like this:

- Encrypt things with the recipient's public key
- Decrypt things with the recipient's private key.

Remember, both keys belong to the same person, so they are the “recipient” in this case.

So if, for example, Bob wants to send Alice a confidential message, Alice must first generate a key pair for herself. Bob then encrypts his message with Alice's public key. Then, holding her private key, only Alice can decrypt the message.

Even if an attacker captures the ciphertext and Alice's public key (which is of course public by definition) they cannot recover the message because they do not have Alice's private key.

In practice, you almost never encrypt large amounts of data directly with a public-key algorithm, since they’re comparatively slow. Instead, the standard pattern (used by e.g. TLS) is called “hybrid encryption”. With hybrid encryption, the parties use asymmetric cryptography to agree on and exchange a short, random session key, and then use that session key with a faster symmetric algorithm (e.g. AES) to protect the actual bulk data. The asymmetric layer is used once, at setup, to protect the symmetric key and solve the insecure channel issue we discussed in the previous post.

This is why understanding the public-key layer is still essential for security operations, even though most of the bytes on the wire will be protected with symmetric encryption.

## Summary

Alright, so now we have an understanding of how to generate asymmetric key pairs and use them for encryption! This helps solve the “bootstrap problem” of agreeing upon shared secrets over an insecure channel.

In the next post, we’ll look at how asymmetric cryptography can help us solve another problem – authentication.

See you in the next post!

## About the author

Hi there! 👋

My name is Braden Wright, and I'm software and security engineer with 5+ years of security engineering experience and more than double that in software. I spend my work days building security infrastructure and tooling, spotting ne'erdowells in SIEM logs, and injecting security into the software development process. I spend my days off writing software and firmware, prototyping hardware boards plus 3D modelling for their enclosures, and trying to "touch grass" in local hiking trails or state parks.

Despite the advent of generative AI, I still enjoy writing longform text and software by hand, key-by-key. So, I'm starting this little blog to document things I've learned in the industry. Hope you enjoy!
