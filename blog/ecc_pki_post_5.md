# ECC Algorithms and Security Levels

In the previous post, we looked at the math underpinning elliptic curve cryptography in general principle. That’s all well and good, but we need a way to actually use them in practice. So, in this post, we’re going to look at real life key exchange and signature algorithms that are actually built on elliptic curves, compare them with the older RSA approach, and talk a little more about general “security levels” as a basis for comparison.

## Cryptography in practice

As you’ll recall from past posts, asymmetric cryptography has two distinct workloads: **authentication** (via signing) and **confidentiality** (key agreement / encryption). The two workloads require using the same keys differently, different algorithms are designed for different jobs, mixing them up is a common source of operational headaches, etc.

## Signatures

### ECDSA

**Elliptic-Curve Digital Signature Algorithm (ECDSA)** is the elliptic-curve equivalent of DSA. It yields a signature `(r, s)` over a hash of the message using the private key, which anyone can verify with the public key. ECDSA is very widely deployed - it underpins many TLS server certificates and code-signing uses, and is especially useful for mobile and embedded devices due to its smaller key size relative to RSA.

One operational footgun to be aware of with ECDSA: it requires a cryptographically-secure random nonce for each signature, and if that nonce is ever reused or weak, the private key can be recovered. Correct, constant-time implementation is important.

### EdDSA / Ed25519

**EdDSA** (Edwards-curve Digital Signature Algorithm) is an even newer, cleaner signature family. The most common, concrete implementation is the **Ed25519** algorithm, which uses a Curve25519-based Edwards curve. Its properties make it a favorite for practical tooling for several reasons:

- **No pre-hashing required.** Ed25519 hashes internally as part of the scheme, so you can sign raw bytes directly. This removes the "did I hash it right?" class of mistakes we talked about in the last post.
- **Deterministic, nonce-safe.** Ed25519 derives its per-signature nonce values deterministically from the key and message, so there is no weak-nonce failure mode the way ECDSA has.
- **Constant-time and simple.** The Edwards-curve addition formula has no edge cases, which makes it much easier to implement in a way that is resistant to side-channel attacks from not using constant time.
- **Fast signing and compact signatures.** Signatures are only 64 bytes, so it’s incredibly lightweight.

## Key Agreement

### ECDH

**Elliptic-Curve Diffie-Hellman (ECDH)** lets two parties, each with their own key pair, derive a shared secret over an insecure channel. We talked about this in an earlier post, but to summarize: Alice computes `d_A · Q_B` and Bob computes `d_B · Q_A`; both arrive at the same point, from which a symmetric session key is derived. This is the workhorse of the TLS handshake's key exchange.

### Curve25519

Just like how ECDH and ECDSA go together, when you need key agreement to pair with Ed25519 signatures, the companion curve is **X25519,** the Diffie-Hellman form of Curve25519. These two algorithms are designed to work together, are part of the same family, and share a lot of the same positives that drive people towards elliptic curve crypto in the first place: speed, lower processing overhead, strong auditable security.

## Security levels revisited

We’ve talked before already about how key length alone isn’t everything. Elliptic curve algos have significantly smaller key lengths than RSA while providing equivalent protections.

It’s important to keep in mind, though, that as of this writing, the best-known attack against these algorithms is a square-root attack. So, the security strength is about half the scalar bit-length.

Relative “security level” strengths shake out roughly like this:

| Algorithm | Key size | Approx. security level |
| --- | --- | --- |
| RSA-2048 | 2048 bits | \~128 bits |
| RSA-3072 | 3072 bits | \~128–140 bits |
| ECDSA P-256 | 256 bits | \~128 bits |
| Ed25519 | 256 bits | \~128 bits |
| X25519 | 256 bits | \~128 bits |

So Ed25519's 256-bit key (i.e. 32 byte key) delivers the same \~128-bit security as RSA-2048's 2048-bit key, but with a fraction of the size and a faster, cleaner signing path.

## NIST curves vs. Ed25519

Historically, much of the ecosystem standardized on NIST-recommended curves such as P-256 (secp256r1), used with ECDSA. These are fine and widely supported, though due to the relationship of NIST to one particular country’s government, there have always been whispers in the cryptography community about deliberate weakening or “backdoors” in the chosen curves or RSA primes published by NIST.

By contrast, Ed25519, comes from a different, more recent design (the "Edwards" and "Curve25519" family). For a long time this meant that it had less universal support in some libraries and devices, but that gap has closed substantially in recent years. These days, Ed25519 is now broadly supported in OpenSSL, SSH, and modern TLS stacks.

For a security operations team choosing what to standardize on, the practical considerations are:

- **Ecosystem support.** Does your toolchain, your CA software, and your API all support the curve/algorithm? Ed25519 support is now broad, but legacy systems may still expect RSA or P-256.
- **Operational simplicity.** Ed25519's no-prehash signing and deterministic behavior reduce the chance of implementation mistakes in your own tooling.
- **Future-proofing.** We haven’t talked about quantum computing yet (e.g. Shor’s algorithm) but we’ll get there! A \~128-bit security level is the current baseline security level, but note that *all* of these classical algorithms - RSA, ECDSA, Ed25519, etc. - are vulnerable to large-scale quantum cryptanalysis. Choosing ECC today does not delay the need for post-quantum schemes.

## Summary

We’ve now looked at the math and associated algorithms underpinning ECC. Although we haven’t looked at OpenSSL CLI invocations directly, you should hopefully now feel relatively comfortable understanding enough of the manpage and CLI options to issue yourself a key pair or to digitally sign something.

Since this blog is meant to focus primarily on security operations, I want to turn our attention next to the infrastructure that distributes and governs these keys: what a PKI actually is, the structure of the X.509 certificate that carries a public key and its identity, the lifecycle of issuance and revocation, etc. These key-pair mechanics, the curve mathematics, and the algorithms (and associated security levels) that we’ve discussed so far are the foundation on which that infrastructure is built.

See you in the next update!

## About the author

Hi there! 👋

My name is Braden Wright, and I'm software and security engineer with 5+ years of security engineering experience and more than double that in software. I spend my work days building security infrastructure and tooling, spotting ne'erdowells in SIEM logs, and injecting security into the software development process. I spend my days off writing software and firmware, prototyping hardware boards plus 3D modelling for their enclosures, and trying to "touch grass" in local hiking trails or state parks.

Despite the advent of generative AI, I still enjoy writing longform text and software by hand, key-by-key. So, I'm starting this little blog to document things I've learned in the industry. Hope you enjoy!
