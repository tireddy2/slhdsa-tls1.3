---
title: "Use of SLH-DSA in TLS 1.3"
abbrev: "Use of SLH-DSA in TLS 1.3"
category: std

docname: draft-reddy-tls-slhdsa-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "TLS"
keyword:
 - SLH-DSA
 - FIPS205
 

author:
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"
 -
    fullname: Timothy Hollebeek
    organization: DigiCert
    city: Pittsburgh
    country: USA
    email: "tim.hollebeek@digicert.com"
 -
    name: John Gray
    org: Entrust Limited
    abbrev: Entrust
    street: 2500 Solandt Road – Suite 100
    city: Ottawa, Ontario
    country: Canada
    code: K2K 3G5
    email: john.gray@entrust.com
 -
    fullname: Scott Fluhrer
    organization: Cisco Systems
    email: "sfluhrer@cisco.com"

normative:
 RFC8446:
 
 TLSIANA: I-D.ietf-tls-rfc8447bis
 I-D.ietf-lamps-x509-slhdsa:
informative:
  RFC5246:
  RFC9261:
  RFC8555:
  FIPS205:
     title: "FIPS 205: Stateless Hash-Based Digital Signature Standard"
     target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.205.pdf 
     date: false
  FIPS180:
     title: "NIST, Secure Hash Standard (SHS), FIPS PUB 180-4, August 2015"
     target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf 
     date: false
  FIPS202:
     title: "NIST, SHA-3 Standard: Permutation-Based Hash and Extendable-Output Functions, FIPS PUB 202, August 2015."
     target: https://nvlpubs.nist.gov/nistpubs/fips/nist.fips.202.pdf 
     date: false
  PQ-TLS-TTLB:
     title: "The impact of data-heavy, post-quantum TLS 1.3 on the time-to-last-byte of real-world connections."
     target: https://www.amazon.science/publications/the-impact-of-data-heavy-post-quantum-tls-1-3-on-the-time-to-last-byte-of-real-world-connections 
     date: false
 
--- abstract

This memo specifies how the post-quantum signature scheme SLH-DSA {{FIPS205}} is used for authentication in TLS 1.3.

--- middle

# Introduction

Stateless Hash-Based Digital Signatures (SLH-DSA) {{FIPS205}} is a quantum-resistant digital signature scheme standardized by the US National Institute of Standards and Technology (NIST) PQC project. 

This memo specifies how SLH-DSA can be negotiated for authentication in TLS 1.3 via the "signature_algorithms" and "signature_algorithms_cert" extensions.

## Conventions and Terminology {#sec-terminology}

{::boilerplate bcp14+}

This document uses terms defined in {{?I-D.ietf-pquip-pqt-hybrid-terminology}}. For the purposes of this document, it is helpful to be able to divide cryptographic algorithms into two classes:

"Asymmetric Traditional Cryptographic Algorithm": An asymmetric cryptographic algorithm based on integer factorisation, finite field discrete logarithms or elliptic curve discrete logarithms, elliptic curve discrete logarithms, or related mathematical problems.

"Post-Quantum Algorithm": An asymmetric cryptographic algorithm that is believed to be secure against attacks using quantum computers as well as classical computers. Post-quantum algorithms can also be called quantum-resistant or quantum-safe algorithms. Examples of quantum-resistant digital signature schemes include ML-DSA and SLH-DSA.

# Applicability of SLH-DSA

Applications that use SLH-DSA need to be aware that the signature sizes of the algorithms specified in this document are generally large. SLH-DSA offers three security levels: 1, 3, and 5, and two parameter variants for each level:

* Small (s) variant, which are optimized for minimal signature size, have signature sizes ranging from 7856 bytes (128-bit) to 29792 bytes (256-bit).

* Fast (f) variant, optimized for faster key generation and signing, have signature sizes ranging from 17088 bytes (128-bit) to 29792 bytes (256-bit). However, they are slower at signature verification.

Despite offering trade-offs between size and performance, all SLH-DSA variants produce significantly larger signatures than traditional signature algorithms. While SLH-DSA increases the size of the TLS 1.3 handshake, its impact on connection performance is minimal in the context of large data transfers, especially over low-loss networks. For instancee, TLS-based protocols are increasingly used to secure long-lived interfaces in critical infrastructure, such as telecommunication networks. In particular, DTLS-in-SCTP has been mandated in 3GPP for interfaces such as N2 that use long-lived TLS connections. 

# SLH-DSA SignatureSchemes Types

SLH-DSA {{FIPS205}} utilizes the concept of stateless hash-based signatures. In contrast to stateful signature algorithms, SLH-DSA eliminates the need for maintaining state information during the signing process. SLH-DSA is designed to sign up to 2^64 messages and it offers three security levels. The parameters for security levels 1, 3, and 5 were chosen to provide the equivalent of AES-128, AES-192, and AES-256 level of security respectively (see Table 2 in Section 10 of {{?I-D.ietf-pquip-pqc-engineers}}). 

This document specifies the use of the SLH-DSA algorithm in TLS at three security levels. Each security level (1, 3, and 5) defines two variants of the algorithm: a small (S) version and a fast (F) version. The small version prioritizes smaller signature sizes, making them suitable for resource-constrained environments IoT devices. Conversely, the fast version prioritizes speed over signature size, minimizing the time required to generate signatures. However, signature verification with the small version is faster than with the fast version. For hash function selection, the algorithm uses SHA-256 ({{FIPS180}}) for security level 1 and both SHA-256 and SHA-512 ({{FIPS180}}) for security levels 3 and 5. Alternatively, SHAKE256 ({{FIPS202}}) can be used across all security levels.

The following combinations are defined in SLH-DSA {{FIPS205}}:

* SLH-DSA-128S-SHA2
* SLH-DSA-128F-SHA2
* SLH-DSA-192S-SHA2
* SLH-DSA-192F-SHA2
* SLH-DSA-256S-SHA2
* SLH-DSA-256F-SHA2
* SLH-DSA-128S-SHAKE
* SLH-DSA-128F-SHAKE
* SLH-DSA-192S-SHAKE
* SLH-DSA-192F-SHAKE
* SLH-DSA-256S-SHAKE
* SLH-DSA-256F-SHAKE

SLH-DSA does not introduce any new hardness assumptions beyond those inherent to its underlying hash functions. It builds upon established foundations in cryptography, making it a reliable and robust digital signature scheme for a post-quantum world. While attacks on lattice-based schemes like ML-DSA are currently hypothetical at the time of writing this document, such attacks, if realized, could compromise their security. SLH-DSA would remain unaffected by these attacks due to its distinct mathematical foundations. This ensures the ongoing security of systems and protocols that use SLH-DSA for digital signatures.

However, ML-DSA outperforms SLH-DSA in both signature generation and validation time, as well as in signature size, making it a recommended choice for end-entity certificates. SLH-DSA, in contrast, offers smaller key sizes but larger signature sizes. Given its well-established hardness assumption, SLH-DSA may be preferred for TLS applications where high confidence in security is a priority, such as for long-lived TLS sessions and deployments where computational costs of signature generation and validation are minor compared to data transmission and processing demands of user data. The findings in {{PQ-TLS-TTLB}} shows that while PQ algorithms increase the TLS 1.3 handshake data size, their effect on connection performance is minimal for large data transfers, especially in low-loss networks. Additionally, SLH-DSA is suitable for use in CA certificates due to its strong cryptographic assurances and smaller key sizes. Its robustness against emerging quantum attacks makes it a dependable choice for trust anchors and long-term security, even though it has larger signature sizes. 

As defined in {{RFC8446}}, the SignatureScheme namespace is used for the negotiation of signature scheme for authentication via the "signature_algorithms" and "signature_algorithms_cert" extensions. This document adds new SignatureSchemes types for the SLH-DSA as follows.

~~~
enum {
  slhdsa_sha2_128s (0x0911),
  slhdsa_sha2_128f (0x0912),
  slhdsa_sha2_192s (0x0913),
  slhdsa_sha2_192f (0x0914),
  slhdsa_sha2_256s (0x0915),
  slhdsa_sha2_256f (0x0916),
  slhdsa_shake_128s (0x0917),
  slhdsa_shake_128f (0x0918),
  slhdsa_shake_192s (0x0919),
  slhdsa_shake_192f (0x091A),
  slhdsa_shake_256s (0x091B),
  slhdsa_shake_256f (0x091C)
} SignatureScheme;
~~~

It is important to note that the slhdsa* entries represent the pure versions of these algorithms and should not be confused with prehashed variant HashSLH-DSA, also defined in {{FIPS205}}.

SLH-DSA supports two signing modes: deterministic and hedged. In the deterministic mode, the signature is derived solely from the message and the private key, without requiring fresh randomness at signing time, this eliminates dependence on an external random number generator (RNG). It instead uses a precomputed randomness embedded in the private key.  The hedged mode incorporates both fresh randomness generated at signing time, as well as the precomputed randomness, thereby offering somewhat stronger assurances. In the context of TLS, authentication signatures are computed over unique handshake transcripts, making each signature input distinct for every session. This property allows the use of either signing mode. The choice between deterministic and hedged modes does not affect interoperability, as the verification process is the same for both. In both modes, the context parameter defined in Algorithm 22 and Algorithm 24 of {{FIPS205}} MUST be set to the empty string.

The signature MUST be computed and verified as specified in {{Section 4.4.3 of RFC8446}}.

The corresponding end-entity certificate when negotiated MUST use id-slh-dsa-sha2-128s, id-slh-dsa-sha2-128f, id-slh-dsa-sha2-192s, id-slh-dsa-sha2-192f, id-slh-dsa-sha2-256s, id-slh-dsa-sha2-256f, id-slh-dsa-shake-128s, id-slh-dsa-shake-128f, id-slh-dsa-shake-192s, id-slh-dsa-shake-192f, id-slh-dsa-shake-256s and id-slh-dsa-shake-256f respectively as defined in {{I-D.ietf-lamps-x509-slhdsa}}}.

The schemes defined in this document MUST NOT be used in TLS 1.2 {{RFC5246}}. A peer that receives ServerKeyExchange or CertificateVerify message in a TLS 1.2 connection with schemes defined in this document MUST abort the connection with an illegal_parameter alert.

# Security Considerations

The security considerations discussed in Section 9 of {{I-D.ietf-lamps-x509-slhdsa}} need to be taken into account. 

SLH-DSA imposes an upper bound of 2^64 signatures per key. If a key pair were used to sign 10 billion messages per second, it would take over 58 years to sign 2^64 messages. While this limit is extremely large, it may need to be considered if a single SLH-DSA private key was shared between a huge number of TLS servers all making extremely frequent negotiations.

## Key Lifetime Management

In order to maintain cryptographic safety in high-scale environments, deployments MUST:

   *  Rotate SLH-DSA certificates and keys based on expected signature usage, ensuring ample margin from the 2^64 signature limit.
   *  Monitor the number of signatures generated per SLH-DSA private key to ensure it   
      remains well below the 2^64 signature limit.

# IANA Considerations

This document requests new entries to the TLS SignatureScheme registry,
according to the procedures in {{Section 6 of TLSIANA}}.

| Value   | Description                        | Recommended | Reference      |
|---------|------------------------------------|-------------|----------------|
| 0x0911  | slhdsa_sha2_128s                   | N           | This document. |
| 0x0912  | slhdsa_sha2_128f                   | N           | This document. |
| 0x0913  | slhdsa_sha2_192s                   | N           | This document. |
| 0x0914  | slhdsa_sha2_192f                   | N           | This document. |
| 0x0915  | slhdsa_sha2_256s                   | N           | This document. |
| 0x0916  | slhdsa_sha2_256f                   | N           | This document. |
| 0x0917  | slhdsa_shake_128s                  | N           | This document. |
| 0x0918  | slhdsa_shake_128f                  | N           | This document. |
| 0x0919  | slhdsa_shake_192s                  | N           | This document. |
| 0x091A  | slhdsa_shake_192f                  | N           | This document. |
| 0x091B  | slhdsa_shake_256s                  | N           | This document. |
| 0x091C  | slhdsa_shake_256f                  | N           | This document. |
--- back

# Acknowledgments
{:numbered="false"}

Thanks to Bas Westerbaan, John Mattsson, D.J. Bernstein, Alicja Kario, and Peter Campbell for the discussion and comments.
