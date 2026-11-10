# Elliptic Curves 101

Up until now, I’ve been doing a lot of hand waving and talking about theory. We sort of looked at how you use asymmetric cryptography in practice to security communications and provide tamper evidence, but we haven’t looked in detail at how any of it actually works.

So, this post is a little heavier on math than the ones before it. We’re going to look at the mathematical underpinnings of a particular type of asymmetric cryptography algorithms: elliptic curve cryptography (ECC).

In a previous post, I alluded to this being my current favorite family of algos, primarily because it’s fast and easy to work with. The smaller key sizes and resultant operation speed-up compared to RSA makes these algorithms a much more efficient choice for modern security infrastructure.

## Real algorithms have curves

An elliptic curve is defined by an equation of the form:

```
y² = x³ + ax + b
```

...over a finite field of numbers. That is, a discrete set of numbers that doesn’t repeat to infinity like the set of all real numbers, for example. You may recognize the shape of this function from junior high algebra class as one that could be graphed into a line or curve of some type. The constants `a` and `b` are chosen such that the curve is "nonsingular", meaning it has no cusps or self-intersections, which is a requirement for the math to work out cleanly.

Operationally, what matters most is not necessarily the picture of the curve, but the set of points that satisfy the equation, together with a way to "add" two points to get a third point on the curve. This collection of points plus the addition operation forms a mathematical group.

## The group operation

Given two points P and Q on the curve, there is a well-defined rule for computing a third point R = P + Q.

1. Draw a straight line through P and Q, which intersects the curve at a third point
2. Reflect that point across the x-axis.

Special rules handle the case where the line is vertical and where you are "adding" a point to itself (known as “point doubling”), like using the tangent line instead.

A few things to note about the group of numbers on the line:

- The set of points plus this addition is **closed**

i.e. the result is always another point on the curve, within the group

- The set of points is always **associative**

i.e. it doesn’t matter what order you add them together in

- The set of points has an **identity element**

i.e. a special point at infinity that leaves elements unchanged when added with it 

- Every point on the line has an **inverse**

i.e. an element that, when added, has the opposite effect

Also, repeated addition is really efficient - you can compute k·P (P added to itself k times) very quickly using double-and-add, an algorithm analogous to exponentiation by squaring.

## The Elliptic Curve “Trapdoor”

The security of ECC rests on a variant of the Discrete Logarithm Problem elliptic curves, the Elliptic Curve Discrete Logarithm Problem (ECDLP). This is the same on-way problem leveraged for RSA and Diffie-Hellman key exchanges.

The problem is:

> Given a base point P and a resulting point Q = k·P, find the scalar k.

Computing Q from k is fast due to the "double-and-add" operation, but there’s no known efficient algorithm for working backwards to find k given only P and Q. “Efficient” in this case meaning “feasible or practical in linear time on current computer hardware, given sufficiently large values for these numbers”.

The best known generic attacks (e.g. Pollard's rho) require on the order of the square root of the group order, i.e. the square root of the number of elements in the number field. This means the security strength is roughly half the bit-length of the scalar k, which is the key. This is a crucial point w.r.t. key sizing and is the reason curve and algorithm strength is compared on the basis of effective *security level* instead of raw key length.

## Why ECC is so efficient

This is, honestly, the main draw for ECC and why it’s significant to security operations. You get a comparable level of security to RSA, but with immensely smaller keys!

The strength of RSA grows with the bit-length of its modulus, and RSA-2048 is now considered the practical minimum for \~128-bit security. Remember that we said that ECC strength is half the length of the scalar k. So, it reaches the same \~128-bit security with a key of only about **256 bits** - that’s an eighth of the size!

The practical implications for security ops within an organization are things like:

- **Smaller keys and signatures.** That means less data to store, transmit, and process. For bandwidth- or compute-constrained systems, links, mobile devices, etc., this is a real win in practice.
- **Faster operations.** Smaller numbers result in less time spent on computations. Handshakes complete faster, signing and verifying are cheaper, etc. This pays off when you’re doing, say, millions of operations like in high-throughput TLS endpoints or automated signing in a build pipeline.
- **Lower resource use.** Since smaller keys are friendlier to constrained hardware and to high-volume server deployments, this makes it easier to justify to the bean counters the idea of “secure-by-default” everywhere. There’s less of a monetary incentive to cut corners.

For a security operations practitioner or system, this efficiency is not just a performance nicety, it actually lowers the operational cost of doing cryptography *correctly* at scale. It’s the exact kind of thing that lets you enforce strong crypto on every endpoint and service rather than leaving pockets of weak or missing cryptography because it was too heavy, costly, or slow.

## The key pair and the curve

So, in ECC:

- The **private** key is a random scalar k.
- The **public** key is the point `Q = k·P`, where P is a fixed publicly-known base point (aka “generator”) of the curve's group.

Like we’ve talked about, deriving Q from k is fast with double-and-add. Recovering k from Q is computationally infeasible due to the ECDLP. The same key pair then permits the two usages I talked about in the previous two posts: encryption/key-agreement to derive shared secrets, and signing to prove authenticity.

## Curve selection

!!! Important !!!

Not every curve is the same - you can’t just pick whatever function you want.

A usable curve has a well-understood, large prime-order subgroup so the ECDLP is actually difficult, standardized parameters so implementations are vetted by proper experts, and a clean, auditable construction. This is where the family of Edwards curves - the “Ed” in “Ed25519” - enters in. Edwards curves have a particularly clean addition formula with no special edge cases, which makes constant-time, side-channel-resistant implementations much easier to write correctly.

## Summary

Not that bad, right? And you were probably worried the math would be boring **😏**

Like a lot of modern cryptography, the math underpinning it is a simple, elegant problem like the DLP. There’s very little heavy-lifting involved w.r.t. actual number crunching and evaluation. Not sure about you, but I’ve written much more complicated algorithms in code than the simple mathematical functions underpinning this stuff!

In the next post, we’ll look at how to apply ECC in practice, and some of the security benefits it yields. Less math, it’ll be more similar to the previous posts.

Stay tuned!

## About the author

Hi there! 👋

My name is Braden Wright, and I'm software and security engineer with 5+ years of security engineering experience and more than double that in software. I spend my work days building security infrastructure and tooling, spotting ne'erdowells in SIEM logs, and injecting security into the software development process. I spend my days off writing software and firmware, prototyping hardware boards plus 3D modelling for their enclosures, and trying to "touch grass" in local hiking trails or state parks.

Despite the advent of generative AI, I still enjoy writing longform text and software by hand, key-by-key. So, I'm starting this little blog to document things I've learned in the industry. Hope you enjoy!
