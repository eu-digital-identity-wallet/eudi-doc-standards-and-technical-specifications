<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification for ZKP Implementation in EUDI Wallet

## Abstract

The present document specifies the technical specification and requirements for Zero-Knowledge Proof (ZKP) Implementation in EUDI Wallet.

### [GitHub discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/discussions/485)


## Licensing and Reuse

© European Union, 2025-2026.

This document is made available under the Creative Commons Attribution 4.0 International licence (CC BY 4.0), unless otherwise stated.

You may reuse this document provided that appropriate credit is given and any changes are indicated.

The full licence text is available at:
https://creativecommons.org/licenses/by/4.0/

## Versioning

| Version | Date | Description |
|---------|------------|------------|
| `0.1` | 2025-04-03 | Initial version for discussion |
| `0.2` | 2025-04-23 | Updates based on first group meeting |
| `0.3` | 2025-05-02 | Updates based on second group meeting |
| `1.0` | 2025-05-21 | Final version |
| `1.0.1` | 2026-01-30 | Editorial update (licensing and reuse clarification) |


## 1 Introduction and Overview
This document provides a technical overview of existing Zero-Knowledge Proof (ZKP) 
systems that could serve as potential candidates for inclusion in a **future** technical 
specification for ZKP implementation within the EUDI  Wallet. At present, no existing 
ZKP approach can be deemed fully suitable or mature enough for direct integration into 
the EUDI Wallet. The objective of this document is therefore to examine current 
ZKP technologies from a technical perspective, assess their applicability in the context 
of the EUDI Wallet ARF, and identify existing gaps or limitations.

High Level Requirements (HLRs) of the ARF with respect to the use of ZKPs are presented
in the Annex 2 of the ARF under Topic 53.

## 2 Available Zero-Knowledge Proof schemes
Following the classification of ETSI TR 119 476 [ETSI_119476] and [Topic_G] we discuss 
two categories of ZKP schemes: multi-message signature schemes and proofs for arithmetic circuits.

## 2.1 Multi-message signature schemes
From a high-level perspective, multi-message signature-based solutions are implemented as follows. 
Initially, a PID Provider or Attestation Provider constructs an attestation that can be 
represented in the form of a list of messages. Then it uses a multi-message signature algorithm 
and generates a (short) signature over the set of messages. A Wallet Unit can now selectively hide 
any of the messages and generate a ZKP which proves that "the Wallet Unit possesses a list of messages
that together with the revealed ones can verify the signature of the PID Provider or Attestation Provider". 

### 2.1.1 BBS+
BBS+ is a digital signature protocol which is used for signing a set of
messages.  It was first envisioned by Boneh, Boyen, Shacham in [BBS2004] (from
where it takes its name), touched and re-visited by [Cam2016]. Currently under
standardisation by the IRTF Crypto Forum Research Group [Loo2025] based on the
improvements presented in [Tes2023], as well as by ISO/IEC in "ISO/IEC PWI 24843 
Privacy-preserving attribute-based credentials". The description of BBS+ in the rest of this
section is based on [Tes2023]. 

#### 2.1.1.1 Issuance
The PID or Attestation Provider transforms a PID or attestation into a list of 
messages (e.g., using a technique such as [Kal2022]). This list of messages is 
then signed using the BBS+ signature scheme as described in [Tes2023], which relies 
on pairing-friendly elliptic curves. The resulting signed attestation is subsequently 
transmitted to the Wallet Unit.

#### 2.1.1.2 Presentation
A Wallet Unit can generate a presentation of an attestation that includes only a 
subset of the messages from the list generated during issuance. It then creates a 
zero-knowledge proof (ZKP), using the scheme described in [Tes2023], demonstrating 
that the disclosed messages are indeed part of the original list of messages 
signed by the Provider.

#### 2.1.1.3 Performance
BBS+ proof generation and verification is reported to be less than 2ms. The size 
of the BBS+ proof is 272+32*i bytes, where i is the number of hidden messages. 

#### 2.1.1.4 Discussion
BBS+ achieves high performance and its building blocks are well-researched. However.
pairing-based cryptographic algorithms and pairing friendly elliptic curves are not
standardized and they are not supported by existing Hardware Security Modules (HSMs),
Secure Elements, or SIM cards.
Similarly, algorithms for transforming attestation formats specified in [ISO/IEC 18013-5] or [SD-JWT VC]
into list of messages are missing. Finally, the BBS+ approach under standardization
does not consider binding of an attestation to a public key controlled by the Wallet Unit
of the User. 

### 2.1.2 Pairing-free BBS schemes
Recent efforts (BBS# [Des2025] and SAAC [Api2025]) propose pairing-free variants
of BBS. These schemes leverage a construction known as Keyed-Verification Anonymous Credentials (KVAC),
which is a type of anonymous credential used in a special setting where the Provider and the
Relying Party are the same entity (i.e., they share a private key). KVACs are very
efficient, do not rely on pairings, and are widely adopted in the Signal messaging system [Cha2020].
The main building block of KVACs is a Message Authentication Code (MAC). Both approaches
build upon the solution presented in [Bar2016] which creates a MAC for a list of
messages using a paring free variant of BBS signatures [BBS2004]  and then allows a user
to reveal a subset of these messages, proving at the same time that it knows
a valid MAC over the complete list of messages; this proof can be verified using
the Provider's secret key.  
Additionally, BBS# proposes a solution for device-binding from ECDSA-signatures, 
relying on re-randomization of ECDSA signatures and public keys. Furthermore, 
a trust model for BBS# that covers revocation and proof of validity is defined
in [BBT2025].

#### 2.1.2.1 Issuance
The PID or Attestation Provider transforms a PID or attestation into a list of 
messages. Then it produces a MAC *A* using its secret key and a random number *e*. Finally,
it transmits (A,e) to the Wallet Unit.

#### 2.1.2.2 Presentation
The Wallet Unit randomizes *A* and obtains a new *A'*. Furthermore using its attestation,
a random number, and  *e* it generates a new number *B'*. *B'* has the property that it equals to 
*A'* raised to the secret key of the Provider. In the setting where the Provider and
the Relying Party are the same entity, the Relying Party has only to check if the
latter equality holds. However, in the general case the Relying Party does not 
have access to the private key of the Provider. Therefore two approaches are proposed:

* The Relying Party asks the Provider to verify that *B'* equals to *A'* raised to
its secret key. Because both numbers are randomized, the Provider cannot linked them
back to User.
* The Wallet Unit asks the Provider to verify *B'* equals to *A'*. This can be done
in batch and at an earlier time. In order to preserve unlinkability, the Wallet Unit
"blinds" these two numbers and the provider generates a proof based on the blinded
values. This is achieved using Oblivious Issuance of Proofs [Orr2024].

Additionally, BBS# defines algorithms for public key randomization with the aid 
of the WSCD, and for proving possession of the corresponding private key, which is
stored securely within the WSCD.

#### 2.1.2.3 Performance
[Des2025] reports for BBS# proof generation and verification times of less than 2.5ms in high-end
mobile devices. Similarly reports that the size of a BBS# presentation proof is 416+U*32
bytes, where U denotes the number of hidden messages. 

#### 2.1.2.4 Discussion
BBS# and SAAC offer high performance but introduce additional communication between 
the Relying Party or Wallet Unit and the Provider. This additional interaction is 
comparable in  volume to batch issuance of attestations. Notably, 
both schemes avoid the use of pairing-based cryptography, however, they 
still require standardization. Furthermore, in both approaches the operations 
required from the Provider side are not currently supported by existing Hardware 
Security Modules (HSMs). Finally, BBS# supports binding an attestation to a public 
key controlled by the Wallet Unit and stored securely within the associated WSCD.  

### 2.1.3 BBS with ECDSA proof of possession 
Recent ZKP schemes enable the proof of possession of a secret key corresponding 
to an ECDSA digital signature, along with a commitment to the associated public key.
These schemes can be constructed using either Sigma protocols [Cel2024] or zkSNARKs
[Woo2025]. Furthermore, constructions such as [Cha2020] enable the generation of a proof
that the committed public key is indeed included in an attestation. 
By integrating those proof mechanisms with BBS signatures, it 
becomes possibly to bind an attestation to a private key stored in a WSCD.

#### 2.1.3.1 Issuance
The PID or Attestation Provider follows the same steps described in section 2.1.1.1
including however in the list of signed message an ECDSA public key.

#### 2.1.3.2 Presentation
The Wallet Unit performs the steps described in section 2.1.1.2 making sure that
the public key included in the list of messages is hidden. It then performs the
following additional steps: 
* it creates a commitment of the public key
* it generates a proof that that the committed public key is the same as the (hidden) public
key included in the attestation (e.g., using [Cha2020])
* it generates an ECDSA digital signature of a fresh nonce using the
device's WSCD
* it generates a proof that signature can be verified using the 
committed public key (e.g., using [Cel2024] or [Woo2025]). 

#### 2.1.3.3 Performance
This approach introduces some additional overhead related to  proofs.
[Cel2024] reports proof size greater than 160kB, and proof generation and verification 
times over 610ms and 450ms respectively in a Macbook M1 with 8 GB of memory. [Woo2025]
reports proof generation time 1.6msec. and proof verification time 1.7msec in a
AMD Ryzen Threadripper 5995WX 1.8GHz CPU, 256 GBs RAM server. Similarly, [Woo2025]
reports proof size equal to 406 bytes. Furthermore, older implementations report
the performance of [Cha2020] to be around 1 sec. in a Samsung Fold smart phone and the 
size of the proof to be over 100kb. 

#### 2.1.3.4 Discussion
The combination of these approaches achieve proof-of-possession of an ECDSA private key
for attestations signed using BBS signatures with pairings (hence support for paring 
cryptography and pairing-friendly curves is required). Such a combination appears to
be still work in progress: its feasibility, performance, 
and security requires 
further investigation.  Finally, these schemes still require standardization.   


## 2.2 Proofs for arithmetic circuits (programmable ZKPs)

These solutions are based on a program expressed in the form of an arithmetic
circuit. This circuit receives a secret input, referred to as the witness,
which can be for example an attestation, as well as a public statement. The
circuit performs a calculation and outputs true if certain conditions hold (e.g.,
“the attestation includes an age attribute with value > 18”). A Wallet Unit
can then generate a ZKP which proves that “the Wallet Unit knows a witness (e.g.,
an attestation), which when provided as input to a certain circuit using the
provided statement, the circuit outputs true”. 

### 2.2.1 Anonymous credentials from ECDSA
Anonymous credentials based on ECDSA are presented in [Fri2024]. This approach 
builds upon the Ligero protocol [Ame2017].

#### 2.2.1.1 Setup
This solution does not require a trusted setup phase. However, the arithmetic 
circuits used to perform cryptographic computations within the Wallet Unit must 
be carefully designed, implemented, and distributed.

#### 2.2.1.2 Issuance
PID Providers or Attestation Providers remain unaware of the use of this scheme; 
therefore, no changes are required to the existing issuance process.

#### 2.2.1.3 Presentation
To generate a zero-knowledge proof for the presentation of an attestation, the Wallet Unit 
first encodes the attestation as private inputs (i.e., witnesses) to a suitable arithmetic 
circuit that represents the desired statement. The circuit also defines any public inputs, 
such as the public key of the Provider. The Wallet Unit then runs the zkSNARK prover algorithm 
over the circuit using the witness and public inputs, producing a succinct proof. This proof 
can be verified by any third party using the public inputs.

#### 2.2.1.4 Performance
The solution in [Fri2024] is reported to generate a proof that an [ISO/IEC 18013-5]
attestation is valid and it includes an age_over_18 attribute in 1.2sec in a
Pixel 6 pro phone. The corresponding verification time is 0.6sec. The size of the proof
is approx 400KB.

#### 2.2.1.5 Discussion
The anonymous credentials from ECDSA approach introduces the highest computational and 
communication overhead among the solutions discussed in this document. Additionally, 
several components of the scheme are not yet standardized. This solution also requires 
the design and deployment of separate arithmetic circuits for each type of proof 
statement (e.g., depending on the format or structure of the attestation)

On the other hand, this solution does not require any change to the credential issuance
process and it can be used with the attestation formats specified in [ISO/IEC 18013-5] or [SD-JWT VC].
Similarly, it supports attestation binding to a public key controlled by the Wallet Unit 
and stored securely within the associated WSCD.

### 2.2.2 Crescent
Crescent is presented in [Paq2024] and builds upon Groth16 [Gro2016] zkSNARK

#### 2.2.1.1 Setup
This solution requires a trusted setup during which a proving key and a verification key 
for a specific arithmetic circuit are generated, which should be distributed to Wallet Units
and Relying Parties respectively. This process requires a one-time generation 
of structured reference strings (SRS), which must be securely discarded afterward to prevent 
compromise. The setup is circuit-specific, meaning a new setup must be performed for 
each unique statement or circuit. Additionally, the arithmetic 
circuits used to perform cryptographic computations within the Wallet Unit must 
be carefully designed, implemented, and distributed.

#### 2.2.2.2 Issuance
PID Providers or Attestation Providers  are oblivious to the use of this scheme, therefore
no changes are required to the existing issuance process.

#### 2.2.2.3 Presentation
In Crescent, the proof generation process is split into two phases: an offline phase 
and an online phase. In the offline phase, which is executed only once, the Wallet Unit 
encodes the attestation as a witness for a pre-defined arithmetic circuit. Using this 
witness the Wallet Unit computes a Groth16 proof. In the online phase, the Wallet Unit 
re-randomizes the proof to ensure unlinkability between multiple presentations of the 
same attestation. This re-randomized proof can then be safely shared with the Relying Party, 
who verifies it using the corresponding verification key and the disclosed public inputs. 

#### 2.2.2.4 Performance
When used in an Intel Xeon W-2133 CPU at 3.60GHz with 6 cores Crescent is reported
to achieve online proof generation in 29.2ms and proof verification in 11.7ms. These numbers
are increased to 315ms and 184ms respectively when device binding is used. The offline
proof generation requires 19sec. The size
of the setup parameters is 580MB and the size of a proof is 1019 bytes. 

#### 2.2.2.5 Discussion
Crescent has better performance than Anonymous credentials from ECDSA solution however
the trusted setup parameters create significant storage overhead. Moreover, some components
of this solution are not standardized. It is highlighted that Crescent relies on Groth16
which relies on pairing-based cryptography. 

Similarly, to anonymous credentials from ECDSA, Crescent does not require any change to the credential issuance
process and it can be used with the attestation formats specified in [ISO/IEC 18013-5] or [SD-JWT VC].
Crescent also supports attestation binding to a public key controlled by the Wallet Unit 
and stored securely within the associated WSCD.



## 3 References

| Reference | Description |
| --- | --- |
| [Ame2017] | Scott Ames, Carmit Hazay, Yuval Ishai,  Muthuramakrishnan Venkitasubramaniam, "Ligero: Lightweight Sublinear Arguments Without a Trusted Setup", in ACM CCS 2017
| [Api2025] | Rutchathon Chairattana-Apirom, Franklin Harding, Anna Lysyanskaya, and Stefano Tessaro, "Server-Aided Anonymous Credentials," available at <https://eprint.iacr.org/2025/513>, 2025
| [Bar2016] | Amira Barki, Solenn Brunet, Nicolas Desmoulins, and Jacques Traor´e, "Improved algebraic MACs and practical keyed-verification anonymous credentials," In Roberto Avanzi and Howard M. Heys, editors, SAC 2016, volume 10532 of LNCS, pages 360–380. Springer, Cham, August 2016
| [BBT2025] | Trust Model : Securing digital identity with advanced cryptographic algorithms, available at https://github.com/Orange-OpenSource/BBS-SHARP-doc-eudi-wallet ,  2025
| [Des2025] | Nicolas Desmoulins, Antoine Dumanois, Seyni Kane, and Jacques Traoré, “Making BBS Anonymous Credentials eIDAS 2.0 Compliant”, Cryptology ePrint Archive, Paper 2025/619, 2025, available at <https://eprint.iacr.org/2025/619> |
| [Topic_G] | Discussion Paper for the European Digital Identity Cooperation Group regarding Topic G: Zero Knowledge Proof, version 1.4 |
| [BBS2004] | Boneh, Dan, Xavier Boyen, and Hovav Shacham. "Short group signatures." In Annual international cryptology conference, pp. 41-55. Berlin, Heidelberg: Springer Berlin Heidelberg, 2004. |
| [Cel2024] | Sofia Celi, Shai Levin, and Joe Rowell, "CDLS: proving knowledge of committed discrete logarithms with soundness," Progress in Cryptology – AFRICACRYPT 2024 |
| [Cha2020] | M Chase, T Perrin, G Zaverucha "The Signal Private Group System and Anonymous Credentials Supporting Efficient Verifiable Encryption." In ACM CCS 2020 |
| [Cha2022] | M Chase, M. Orrù, T Perrin, G Zaverucha "Proofs of discrete logarithm equality across groups." Cryptology ePrint Archive, Paper 2022/1593, 2022, available at <https://eprint.iacr.org/2022/1593>  |
| [ETSI\_119476] | ETSI TR 119 476 V1.2.1, Electronic Signatures and Trust  Infrastructures (ESI); Analysis of selective disclosure and zero-knowledge proofs applied to Electronic Attestation of Attributes |
| [Fri2024] | Matteo Frigo and abhi shelat, Anonymous credentials from ECDSA, Cryptology ePrint Archive, Paper 2024/2010, 2024, available at <https://eprint.iacr.org/2024/2010> |
| [Gro2016] | Jens Groth, “On the Size of Pairing-Based Non-Interactive Arguments”, in EUROCRYPT 2016 |
| [Kal2022]| V Kalos, GC Polyzos, "Requirements and Secure Serialization for Selective Disclosure Verifiable Credentials", in IFIP SEC 2022
| [Loo2025] | Tobias Looker, Vasilis Kalos, Andrew Whitehead and  Mike Lodder, "The BBS Signature Scheme," available at <https://datatracker.ietf.org/doc/draft-irtf-cfrg-bbs-signatures/>, 2025
| [Orr2024] | Michele Orrù, Stefano Tessaro, Greg Zaverucha, Chenzhi Zhu, "Oblivious issuance of proofs", In Annual International Cryptology Conference, 2024 |
| [Paq2024] | Christian Paquin, Guru-Vamsi Policharla, and Greg Zaverucha, "Crescent: Stronger Privacy for Existing Credentials, Cryptology ePrint Archive, Paper 2024/2013, 2024, available at <https://eprint.iacr.org/2024/2013> |
| [Tes2023] |Tessaro, S. and C. Zhu, "Revisiting BBS Signatures", In EUROCRYPT, 2023|
| [Woo2025] | Anna P. Y. Woo, Alex Ozdemir, Chad Sharp, Thomas Pornin, and Paul Grubbs, "Efficient proofs of possession for legacy signature,". IEEE Security and Privacy, 2025 |


