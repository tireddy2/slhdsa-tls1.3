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

# SLH-DSA SignatureSchemes Types

SLH-DSA utilizes the concept of stateless hash-based signatures. In contrast to stateful signature algorithms, SLH-DSA eliminates the need for maintaining state information during the signing process. SLH-DSA is designed to sign up to 2^64 messages and it offers three security levels. The parameters for each of the security levels were chosen to provide 128 bits of security, 192 bits of security, and 256 bits of security. This document specifies the use of the SLH-DSA algorithm in TLS at three security levels. It includes the small (S) or fast (F) versions of the algorithm. For security level 1, SHA-256 ({{FIPS180}}) is used. For security levels 3 and 5, SHA-512 ({{FIPS180}}) is used. SHAKE256 ({{FIPS202}}) is applicable for all security levels. 

The small version prioritizes smaller signature sizes, making them suitable for resource-constrained environments IoT devices. Conversely, the fast version prioritizes speed over signature size, minimizing the time required to generate signatures. However, signature verification with the small version is faster than with the fast version.

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

SLH-DSA does not introduce any new hardness assumptions beyond those inherent to its underlying hash functions. It builds upon established foundations in cryptography, making it a reliable and robust digital signature scheme for a post-quantum world. While attacks on lattice-based schemes like ML-DSA can compromise their security, SLH-DSA will remain unaffected by these attacks due to its distinct mathematical foundations. This ensures the ongoing security of systems and protocols that use SLH-DSA for digital signatures.

However, ML-DSA outperforms SLH-DSA in both signature generation and validation time, as well as in signature size, making it a recommended choice for end-entity certificates. SLH-DSA, in contrast, offers smaller key sizes but larger signature sizes. Given its well-established hardness assumption, SLH-DSA may be preferred for TLS applications where high confidence in security is a priority, such as for long-lived TLS sessions and deployments where computational costs of signature generation and validation are minor compared to data transmission and processing demands of user data. The findings in {{PQ-TLS-TTLB}} shows that while PQ algorithms increase the TLS 1.3 handshake data size, their effect on connection performance is minimal for large data transfers, especially in low-loss networks. Additionally, SLH-DSA is suitable for use in CA certificates due to its strong cryptographic assurances and smaller key sizes. Its robustness against emerging quantum attacks makes it a dependable choice for trust anchors and long-term security, even though it has larger signature sizes. 

As defined in {{RFC8446}}, the SignatureScheme namespace is used for the negotiation of signature scheme for authentication via the
"signature_algorithms" and "signature_algorithms_cert" extensions. This document adds new SignatureSchemes types for the SLH-DSA as follows.

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

SLH-DSA supports two signing modes: deterministic and hedged. In the deterministic mode, the signature is derived solely from the message and the private key, without requiring fresh randomness at signing time. While this eliminates dependence on an external random number generator (RNG), it may increase susceptibility to side-channel attacks, such as fault injection. The hedged mode mitigates this risk by incorporating both fresh randomness generated at signing time and precomputed randomness embedded in the private key, thereby offering stronger protection against such attacks. In the context of TLS, authentication signatures are computed over unique handshake transcripts, making each signature input distinct for every session. This property allows the use of either signing mode. Therefore, the hedged signing mode can be leveraged to provide protection against  side-channel attacks. The choice between deterministic and hedged modes does not affect interoperability, as the verification process is the same for both. In both modes, the context parameter defined in Algorithm 23 of {{FIPS205}} MUST be set to the empty string.

The signature MUST be computed and verified as specified in {{Section 4.4.3 of RFC8446}}.

The corresponding end-entity certificate when negotiated MUST use id-slh-dsa-sha2-128s, id-slh-dsa-sha2-128f, id-slh-dsa-sha2-192s, id-slh-dsa-sha2-192f, id-slh-dsa-sha2-256s, id-slh-dsa-sha2-256f, id-slh-dsa-shake-128s, id-slh-dsa-shake-128f, id-slh-dsa-shake-192s, id-slh-dsa-shake-192f, id-slh-dsa-shake-256s and id-slh-dsa-shake-256f respectively as defined in {{I-D.ietf-lamps-x509-slhdsa}}}.

The schemes defined in this document MUST NOT be used in TLS 1.2 {{RFC5246}}. A peer that receives ServerKeyExchange or CertificateVerify message in a TLS 1.2 connection with schemes defined in this document MUST abort the connection with an illegal_parameter alert.

# Security Considerations

The security considerations discussed in Section 9 of {{I-D.ietf-lamps-x509-slhdsa}} need to be taken into account. 


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
