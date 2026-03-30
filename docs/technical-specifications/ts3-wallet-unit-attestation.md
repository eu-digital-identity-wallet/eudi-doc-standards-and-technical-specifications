<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification of Wallet Unit Attestations (WUA) used in issuance of PID and Attestations

## Abstract

The present document specifies how Wallet Unit Attestations (WUAs) -- comprising Wallet Instance Attestations (WIAs) and Key Attestations (KAs) -- are used in connection with PID Providers and Attestation Providers.

### [GitHub discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/discussions/450)

## Licensing and Reuse

© European Union, 2025-2026.

This document is made available under the Creative Commons Attribution 4.0 International licence (CC BY 4.0), unless otherwise stated.

You may reuse this document provided that appropriate credit is given and any changes are indicated.

The full licence text is available at:
https://creativecommons.org/licenses/by/4.0/

## Versioning

| Version | Date       | Description                        |
|---------|------------|------------------------------------|
| `0.1`   | 2025-03-28 | Initial version for first discussions. |
| `0.2`   | 2025-04-14 | Improvements after first round of feedback and improved scoping. |
| `0.3`   | 2025-04-28 | Addition of Wallet App Attestation, improvements after second round of feedback. |
| `0.4`   | 2025-05-16 | Improvements after commenting. |
| `1.0`   | 2025-08-20 | Changes to WUA Use Cases section (removed references to presentation WUA). |
| `1.1`   | 2025-11-05 | Editorial changes, rename "Wallet App Attestation (WAA)" to "Wallet Instance Attestation (WIA)", remove the "ephemeral WIA", and add attestation proof type. |
| `1.2`   | 2025-11-11 | Editorial changes, included requirements for WUAs related to keystore(s). |
| `1.3`   | 2025-11-25 | Added option to let PID and Attestation Providers communicate WUA expiration preferences to Wallet Unit. |
| `1.4`   | 2026-01-19 | Removed requirements for non-device-bound attestations, added explicit way of communicating WUA requirements, update of examples, editorial changes, added information about exportability of private keys, specified signature algorithms. |
| `1.4.1` | 2026-01-30 | Editorial update (licensing and reuse clarification) |
| `1.5`   | 2026-03-15 | Introduced `client_status` object in the WIA containing `status` (Wallet Instance revocation reference) and `exp` (revocation maintenance expiry). Introduced `key_storage_status` object in the WUA (replacing previous overload of `exp` and `status`) containing `status` (WSCD/keystore revocation reference, with two index assignment options -- see below) and `exp` (revocation maintenance expiry). Added optional per-issuer WIA `client_status.status` entry reuse. Removed `iss` from both WIA and WUA; Wallet Provider identity is now inferred from the signing certificate in the `x5c` JOSE header parameter. Removed `storage_type` and `keys_exportable` from WUA. Removed `kid` requirement for the `jwt` PoP header; Wallet Units SHALL sign the `jwt` PoP with the key at index 0 of `attested_keys` and Issuers SHALL verify under that key. Removed `general_info` from both WIA and WUA; Wallet Solution identity is conveyed via `wallet_name`, `wallet_version`, and `wallet_link` in the WIA. Introduced `wallet_solution_certification_information` as a top-level WIA claim. Replaced `storage_certification_information` with the `certification` field defined in Appendix D of OID4VCI. Renamed `preferred_ttl` to `preferred_key_storage_status_period` and introduced `preferred_client_status_period`; both constrain the remaining revocation maintenance period (`client_status.exp` or `key_storage_status.exp` minus current time) rather than `exp`. Added `wallet_version` (REQUIRED) to the WIA; repurposed `wallet_link` as a RECOMMENDED URI for further information about the Wallet Solution. Removed appendix discussing different revocation options. Renamed 'Wallet Unit Attestation (WUA)' to 'Key Attestation (KA)'; repurposed 'WUA' as the umbrella term covering both WIAs and KAs. Introduced two options for `key_storage_status.status` index assignment in KAs: (1) type-shared index, where all Wallet Units using the same WSCD or keystore type share a single status list index (existing behaviour); (2) per-KA index, where each KA is assigned its own fresh index representing the revocation state of the Wallet Unit's WSCD or keystore instance attested in that KA, with an optional per-issuer reuse sub-option analogous to the WIA `client_status.status` per-issuer reuse option. |

## 1 Introduction and Overview

The present document specifies Wallet Unit Attestations (WUAs) for actors in the EUDI Wallet ecosystem, ensuring interoperability across the ecosystem. WUAs comprise two types: Wallet Instance Attestations (WIAs) and Key Attestations (KAs). WUAs are used exclusively during the issuance of PIDs and attestations, and the mechanisms specified here are designed to be compatible with OID4VCI. The goal of the specification is that the solution is technically simple and does not introduce unnecessary complexity.

[Section 2](#2-solution-description) of this document will serve as the contents of the technical specification and the appendix provides examples and a mapping to ETSI TS 119 602.

### 1.1 WUA Use Cases

WUAs allow information to be transferred from the Wallet Provider (via the Wallet Unit) to PID Providers and Attestation Providers. WUAs comprise two types:

* *WIAs*: Wallet Instance Attestations containing attested information about the Wallet Instance (i.e., the Wallet application). WIAs allow PID Providers and Attestation Providers to verify the integrity and authenticity of the Wallet Instance and to revoke their credentials (if needed) in case a Wallet Provider revokes a Wallet Instance.

* *KAs*: Key Attestations containing attested information about the security of cryptographic keys stored in the Wallet Unit. KAs allow PID Providers and Attestation Providers to determine the security level of the keys to which they are binding PIDs or attestations, and to revoke their credentials (if needed) in case a security vulnerability affecting the WSCD or keystore is identified, or -- when the Wallet Provider uses a per-KA index -- upon user request.

### 1.2 Scope

This STS specifies the following:

* The *transfer* of WIAs and KAs between the Wallet Unit and the issuing party (i.e., either PID or Attestation Provider).
* The *format* of WIAs and KAs including their encoding and integrity protection mechanism.
* The *content* of WIAs and KAs.
* The *life cycle* of WIAs and KAs.
* The *revocation mechanism* for WIAs and KAs.

> Note that *how* Wallet Providers issue WIAs and KAs to the Wallet Unit is out of scope for this technical specification, as this will only be done by the Wallet Unit providers themselves and does therefore not require any standard to achieve interoperability.

### 1.3 Requirements Notation

The key words "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119) [RFC8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they are written in all capital letters.

# 2 Solution Description

EUDI Wallet Units shall use two types of Wallet Unit Attestations (WUAs) in relation to issuance: 1) A *Wallet Instance Attestation* (WIA) that attests *only* the integrity of the Wallet Instance (i.e, the app) and 2) a *Key Attestation* (KA) that attests the security of keys stored in the Wallet Unit. A KA describes either a WSCD or a keystore that can be used to securely store cryptographic keys.

The WIA allows PID Providers and Attestation Providers to protect their endpoints by only communicating with Wallet Instances whose integrity is ensured by the respective Wallet Provider, and to revoke their credentials in case a Wallet Provider revokes a Wallet Instance. KAs, on the other hand, allow PID Providers and Attestation Providers to ensure that they only issue PID and attestations that are cryptographically bound to keys that are properly protected (i.e., in a WSCD or a keystore with sufficiently high attack resistance), and to revoke their credentials in case a security vulnerability affecting the keystore or WSCD is identified, or -- when the Wallet Provider uses a per-KA index -- upon user request. WIAs will be used both during issuance of device-bound and non-device-bound attestations, whereas KAs will only be used for device-bound attestations.

A high level overview of the solution is given in the table below:

| **Conceptual Part** | **Solution** |
|---------------------|----------------------------------------------------------------------------------|
| Format              | Both WIA and KA shall be JSON Web Tokens (JWTs) signed by the Wallet Provider. See [Section 2.1 Format](#21-format).   |
| Transport           | The WIA shall be a client attestation sent to a token endpoint at an Authorization Server.  The KA shall be a `key_attestation` element sent within the Credential Request of OID4VCI (either as an `attestation` proof type or in the header of a `jwt` proof type). The `key_attestation` element is extended with EUDI Wallet-specific claims.  See [Section 2.2 Transport](#22-transport).   |
| Content             | The WIA shall contain only very basic information about the application. Some of the information specified in the ARF for the KA will be transferred in existing elements of the OID4VCI protocol, e.g., revocation information and the public key corresponding to a private key stored in the WSCD or keystore. Additional EUDI Wallet-specific claims are added to both the WIA and the KA. The concrete contents are discussed in [Section 2.3 Content](#23-content). |
| Life Cycle          | The WIA shall have a short time-to-live (less than 24 hours). The KA MAY have a longer validity period. The WIA includes a `client_status` object and the KA includes a `key_storage_status` object; the `exp` sub-field of each specifies how long the Wallet Provider guarantees to maintain revocation status at the referenced status list index. `exp` (the token-level claim) denotes only the technical expiration of the token. See [Section 2.4 Life Cycle](#24-life-cycle). |
| Revocation          | Both WIA and KA use Status Lists for revocation of the underlying objects. The WIA `client_status.status` entry conveys the revocation state of the Wallet Instance (optionally scoped per Wallet Instance-Attestation Provider relationship, with reuse of the index value; see Section 2.5). The KA `key_storage_status.status` entry conveys either the revocation state of the WSCD or keystore type, with all KAs of the same type sharing one index (Option 1: type-shared), or the revocation state of the individual Wallet Unit's WSCD or keystore instance (Option 2: per-KA index, with optional per-issuer reuse). See [Section 2.5 Revocation](#25-revocation). |
| Signature Algorithms | ES256, ES384, and ES512 are supported for signing WIAs, KAs, proof-of-possessions, and Token Status Lists. See [Section 2.6 Signature Algorithms](#26-signature-algorithms). |

Below we present details of the technical specification for WUAs.

## 2.1 Format

A Wallet Instance Attestation SHALL be a JSON Web Token (JWT) as specified in [RFC7519](https://datatracker.ietf.org/doc/html/rfc7519).

A Key Attestation SHALL be a JSON Web Token (JWT) as specified in [RFC7519](https://datatracker.ietf.org/doc/html/rfc7519).

## 2.2 Transport

The ARF specifies that [OIDF OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3) is to be  used for issuance, hence the transport of the WIA and KA must be compatible with the options of the OID4VCI protocol.
Note that OID4VCI uses Authorization Server Metadata and Credential Issuer Metadata to specify the need for WIA and KA, respectively.

### 2.2.1 Transport of WIA

The WIA SHALL be an Wallet Attestation as specified in [Appendix E of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3) and sent to the Authorization Server in the Pushed Authorization Request and the Token Request.

A WIA SHALL be used both during issuance of device-bound attestations and non-device-bound attestations.

The WIA SHALL be signed by the Wallet Provider.

The WIA SHALL  be sent along with a Proof-of-Possession (PoP), as specified in [Appendix E of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3).

> Note that the WIA is a mechanism which PID and Attestation Providers can use to protect their endpoints.

#### 2.2.1.1 Wallet Providers Responsibilities for Transport of WIAs

A Wallet Provider SHALL verify the integrity of the Wallet Instance before signing a WIA.

When a WIA is sent to the Wallet Unit, it SHALL have a time-to-live of less than 24 hours. i.e., the difference between expiration time `exp` and the time the Wallet Provider verified the integrity of the Wallet Instance SHALL be less than 24 hours.

The Wallet Provider SHALL ensure that a Wallet Unit has WIAs as needed for issuance.

A Wallet Provider SHALL ensure that their Wallet Units sends the same WIA to at most one PID Provider or Attestation Provider. If a Wallet Provider uses the "per-issuer reuse" option (see [Section 2.5.1](#2.5.1-wia-revocation-and-optional-per-issuer-status-entry-reuse)), a Wallet Unit MAY send a WIA to the same PID Provider or Attestation Provider multiple times. Otherwise, a Wallet Unit SHALL use a WIA in at most one credential issuance process.


> This is to prevent issuer linkability.

#### 2.2.1.2 PID Providers and Attestation Providers Responsibilities for Transport of WIAs

When a PID Provider or Attestation Provider receives a WIA, then they SHALL verify that the signature of the WIA verifies under the public key in the signing certificate included in the `x5c` parameter in the JOSE header of the WIA (i.e., the wallet attestation as specified in Appendix E of OID4VCI). The PID Provider or Attestation Provider SHALL moreover verify that this signing certificate can be verified with a trust anchor found on the Trusted List for Wallet Providers, potentially using intermediate certificates included in the `x5c` parameter.

When a PID Provider or Attestation Provider receives a WIA, then they SHALL check that it has not expired.

When a PID Provider or Attestation Provider receives a WIA, then they SHALL check the signature of the PoP verifies under the public key present in the `cnf`.

### 2.2.2 Transport of Key Attestation (KA)

The KA SHALL be a `key_attestation` as defined in [Appendix D of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3) extended with additional EUDI Wallet-specific claims as specified in [Section 2.3 Content](#23-content).

During issuance of device-bound attestations, a Wallet Unit SHALL include the KA in a proof of type `jwt` or `attestation` in the `proofs` field of their Credential Request to the Credential Issuer.

> Note the `jwt` acts as a proof-of-possession (PoP) of the keys; the Wallet Unit signs a nonce provided by the PID Provider or Attestation Provider during the issuance process. In contrast, if the proof type is `attestation`, there is no PoP. Instead the nonce  is included in the KA itself. This requires that the Wallet Unit sends the nonce to the Wallet Provider, which then immediately issues the KA to the Wallet Unit.

#### 2.2.2.1 Wallet Providers Responsibilities for Transport of KAs

The KA (i.e., the `key_attestation` element in a `jwt` or `attestation` proof type) SHALL be generated and signed by the Wallet Provider.

The Wallet Provider SHALL verify that the keys attested to in the KA are stored in the WSCD or key store as specified in the KA before signing a KA.

The `attested_keys` element of an `key_attestation` SHALL contain at least one key, but MAY contain multiple keys (in order to support batch issuance). The number of keys in the KA sent to a PID Provider or Attestation Provider SHOULD NOT exceed the maximum batch size specified by that Attestation Provider or PID Provider in the Credential Issuer metadata for this type of credential; see ETSI TS 119 472-3, parameter `credential_configurations_supported.credential_metadata.credential_reuse_policy.options.batch_size`.

NOTE: If the number of keys exceeds the batch size, the PID Provider or Attestation Provider will not use all keys in the KA to bind PIDs or attestations to. Since a Wallet Unit uses a KA at most once, this implies that the unused keys will never be used. It is up to the Wallet Provider to devise a strategy for minimizing the number of keys that will remain unused, if needed.

If a Wallet Unit includes a KA in a `jwt` element, then it SHALL be signed by the Wallet Unit with the key at index 0 of the `attested_keys` array within the `key_attestation` object.

> Requiring only one signature on the entire `jwt` may improve the user experience as some WSCDs may require a user gesture to sign. Note that this does not degrade the security as the signature of the Wallet Provider still binds all keys included in the KA to the same WSCD or keystore.

Wallet Providers SHALL ensure that their Wallet Units use a single KA at most once and each public key corresponding to a private key stored in the wallet secure cryptographic device or keystore shall only be included at most in one KA.

> This is to prevent verifier linkability. Note that the attested keys in a KA cannot be re-issued.

A Wallet Unit SHALL send a KA to a PID Provider or Attestation Provider during issuance of device-bound attestations.

A Wallet Unit SHALL NOT send a KA to an Attestation Provider during issuing of non-device-bound attestations.

#### 2.2.2.2 PID Providers and Attestation Providers Responsibilities for Transport of KAs

A PID Provider or an Attestation Provider issuing device-bound attestations SHALL indicate in the `proof_types_supported` parameter in its Issuer Credential Metadata (see [Section 12.2.4 OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3)) that it supports both the `jwt` and the `attestation` proof types for KAs, both of which SHALL include the `key_attestation_required` object.

An Attestation Provider issuing non-device-bound attestations SHALL omit the `proof_types_supported` and the `cryptographic_binding_methods_supported` parameters in the Credential Issuer Metadata.

If a PID Provider or Attestation Provider receives a KA in a `jwt` or an `attestation`proof type, then they SHALL verify that the signature of the KA verifies under the public key in the signing certificate included in the `x5c` parameter in the JOSE header of the KA (i.e., the key attestation as specified in Appendix D.1 of OID4VCI). The PID Provider or Attestation Provider SHALL moreover verify that this signing certificate can be verified with a trust anchor found on the Trusted List for Wallet Providers, potentially using intermediate certificates included in the `x5c` parameter.

If a PID Provider or Attestation Provider receives a KA in a `jwt` proof type, then they  SHALL verify that the signature of the `jwt` element verifies under the key at index 0 of the `attested_keys` array within the `key_attestation` object included in the `jwt`.

If a PID Provider or Attestation Provider receives a KA in a `jwt` proof type, then they SHALL verify that the `jwt` element includes a valid `c_nonce` from their `nonce_endpoint` in the `nonce` field of the `jwt`. Similarly, if a PID Provider or Attestation Provider receives a KA in a `attestation` proof type, they SHALL verify that the `key_attestation` object includes a valid `c_nonce` from their `nonce_endpoint`.

## 2.3 Content

### 2.3.1 Content of WIA

The content of the WIA is specified in [Appendix E of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3). In addition to the attributes required by that specification, the WIA SHALL also include:

- The `wallet_name` claim as specified in Appendix E of OID4VCI. The value of this claim shall be the identifier of the Wallet Solution, as it can be found on the Wallet Provider Trusted List.
- The `wallet_version` claim, introduced by this specification. The value of this claim shall be the version of the Wallet Solution.
- The `wallet_solution_certification_information` claim, introduced by this specification. The value of this claim shall contain information about the conformity assessment body that certified the Wallet Solution, the applicable certification number, and other relevant certification details.
- The `client_status` claim, introduced by this specification. This object contains two sub-fields:
  - `status`: a status list reference as specified in Appendix E of OID4VCI. The value represents the revocation state of the Wallet Instance. See [Section 2.5](#25-revocation) for details.
  - `exp`: a NumericDate (as defined in RFC 7519) specifying the time until which the Wallet Provider commits to maintaining the revocation status at the status list index referenced in `status`. See [Section 2.4.1](#241-status-maintenance-period).

The WIA SHOULD also include the `wallet_link` claim as specified in Appendix E of OID4VCI. The value of this claim shall be a URI where further information about the Wallet Solution can be obtained.

> **Note:** The `client_status.status` entry in the WIA represents the revocation state of the Wallet Instance, not merely the revocation state of the WIA itself. As described in [Section 2.5.1](#251-wia-revocation-and-optional-per-issuer-status-entry-reuse), a Wallet Provider can decide to scope each WIA to a specific PID Provider or Attestation Provider, meaning that all WIAs sent to a specific PID Provider or Attestation Provider contain the same index value.

A PID Provider that has issued a PID to this Wallet Unit SHALL check this `client_status.status` entry at least once every 24 hours for the validity period of the PID; see [Section 2.4.3](#243-pid-provider-and-attestation-provider-responsibilities). An Attestation Provider MAY choose to do the same.

> As the certification scheme has not yet been defined, the exact content of `wallet_solution_certification_information` is undefined. This content will be defined in a future update. Note that this field is subject to alignment with the specification for Trusted Lists once it is released.

### 2.3.2 Content of Key Attestation (KA)

A Wallet Provider SHALL provide a Wallet Unit with different KAs for the Wallet Unit's WSCD and for each of its keystores.

The high-level requirements of the ARF require a number of different attributes being transferred as part of the WIA and KA. Some of these attributes are already defined by the OID4VCI specification, in which case OID4VCI will be used. In particular, the `key_storage` attribute of the `key_attestation` SHALL be used to indicate the 'attack potential resistance' of the place where the attested keys are stored in a KA.

> For a KA about a WSCD, the `key_storage` and `user_authentication` attributes shall be `iso_18045_high` as the WSCD by definition must ensure LoA High.
>
> The EUDI-specific claims are related to the content of the Wallet Provider Trusted List. This content specification must be updated when the content of the Trusted List has been defined. In particular, the identity information in the WIA (conveyed via `wallet_name` and `wallet_link` as specified in Appendix E of OID4VCI, and the Wallet Provider identity inferred from the signing certificate) must be aligned with the Trusted List, such that the information is uniquely linked to an entry on the Trusted List.

The content of the KA is given by the `key_attestation` definition specified in [Appendix D of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3). If a KA is sent as an attestation proof type, then it SHALL also include a `c_nonce` as specified in [Appendix F.3 of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3).

In addition to the required attributes in [Appendix D of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3), the KA SHALL also include:

- The `certification` field of the `key_attestation` element as defined in Appendix D of OID4VCI, providing information about the certification achieved by the WSCD or keystore (e.g., the scheme such as Common Criteria or GlobalPlatform, the evaluated requirements such as the applicable Protection Profile, and the evaluation level). It shall be possible to determine from this field whether the key storage is a WSCD.
- The `key_storage_status` claim, introduced by this specification. This object contains two sub-fields:
  - `status`: a status list reference as specified in Appendix D.1 of OID4VCI. The value represents either the revocation state of the WSCD or keystore *type* used to store the attested keys, or -- under the per-KA index option -- the revocation state of the individual Wallet Unit's WSCD or keystore instance. See [Section 2.5.2](#252-ka-revocation) for the available index assignment options.
  - `exp`: a NumericDate (as defined in RFC 7519) specifying the time until which the Wallet Provider commits to maintaining the revocation status at the status list index referenced in `status`. See [Section 2.4.1](#241-status-maintenance-period).

> **Note:** Information identifying the Wallet Solution (Wallet Provider identifier, Wallet Solution identifier, version, and information URI) is not repeated in the KA; it is conveyed in the corresponding WIA via `wallet_name`, `wallet_version`, and `wallet_link`, with the Wallet Provider identity inferred from the signing certificate.
>
> **Note on the `idx` value of the KA `key_storage_status.status` entry:** When the type-shared index option is used (see [Section 2.5.2](#252-ka-revocation)), all KAs for the same WSCD or keystore type share the same status list index, so the `idx` value is not unique per Wallet Unit. When the per-KA index option is used, the `idx` value is unique to the KA (or unique per issuer when per-issuer reuse is applied). In all cases, PID Providers or Attestation Providers SHALL NOT use the KA `idx` as a Wallet Unit identifier. Instead, the WIA `client_status.status` entry continues to serve as an identifier of the Wallet Instance (and thus the Wallet Unit).
>
> Note that the `key_attestation` element contains the signed keys in `jwk` format. The `jwk` format specifies that each key must contain an algorithm type. Hence, this provides protection against downgrade attacks for attested keys.
>
> Note that `iss` is not required in either the WIA or KA; the Wallet Provider identity can be inferred from the signing certificate included in the `x5c` header parameter of the JOSE header.

During PID issuance, the PID Provider SHALL ensure that the PID they issue is bound to a key originating from a KA whose key storage is a WSCD.

## 2.4 Life Cycle

This section specifies the life cycle of WIAs and KAs, including the status maintenance expiry sub-field (`exp`) within the `client_status` and `key_storage_status` objects.

To allow PID Providers and Attestation Providers to communicate their preferences for the remaining status maintenance period of WIAs and KAs, this specification defines two fields for the Credential Issuer Metadata endpoint (see [Section 12.2.2 of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3)):

* `preferred_client_status_period`: OPTIONAL. An integer specifying a PID or Attestation Provider's preference for the remaining status maintenance period (`client_status.exp` minus current time) of the WIA it receives during issuance, in seconds. This field is placed at the top level of the Credential Issuer Metadata (not within `key_attestations_required`). See [Section 2.4.2](#242-wallet-provider-responsibilities) for how the Wallet Unit and Wallet Provider must handle this parameter.

The following field is to be placed within the `key_attestations_required` object as specified in [Section 12.2.4 of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3):

* `preferred_key_storage_status_period`: OPTIONAL. An integer specifying a PID or Attestation Provider's preference for the remaining status maintenance period (`key_storage_status.exp` minus current time) of the KA it receives during issuance, in seconds. See [Section 2.4.2](#242-wallet-provider-responsibilities) for how the Wallet Unit and the Wallet Provider must handle this parameter.

### 2.4.1 Status Maintenance Period

The WIA SHALL include a `client_status` object and the KA SHALL include a `key_storage_status` object. Each of these objects contains a `status` sub-field (the status list reference) and an `exp` sub-field (a NumericDate as defined in RFC 7519) specifying the time until which the Wallet Provider commits to maintaining the revocation status at the status list index referenced in `status`.

The `exp` sub-field within `client_status` and `key_storage_status` is independent of the token-level `exp` claim. The token-level `exp` denotes when the token itself expires and SHALL NOT be interpreted as the end of the revocation maintenance period. A token with a token-level `exp` in the near future can nonetheless carry a `client_status.exp` or `key_storage_status.exp` far in the future, allowing issuers to rely on the Wallet Provider's revocation infrastructure for as long as PIDs or attestations issued using that token are valid.

### 2.4.2 Wallet Provider Responsibilities

A Wallet Provider SHALL choose the technical validity period of the KA and SHALL maintain the revocation mechanism, see [Section 2.5](#25-revocation), until `key_storage_status.exp` has passed.

A Wallet Provider SHALL include a `client_status` object in every WIA it issues, with a `status` sub-field referencing a status list entry that represents the revocation state of the Wallet Instance. See [Section 2.5](#25-revocation) for the optional per-issuer reuse of this entry.

A Wallet Provider SHALL include a `client_status` object in every WIA it issues and a `key_storage_status` object in every KA it issues.

A Wallet Provider SHALL ensure that a Wallet Unit can always present WIAs and KAs whose `client_status.exp` and `key_storage_status.exp` (respectively) are at least 31 days in the future at the time of presentation to a PID Provider or Attestation Provider. This guarantees that PID Providers can rely on revocation chaining without being forced to issue short-lived PIDs.

A Wallet Provider SHALL ensure that the Wallet Unit fetches the Credential Issuer Metadata during issuance. If a `preferred_key_storage_status_period` field is included in this, then the Wallet Unit SHALL send the KA available to them with (`key_storage_status.exp` - current time) - `preferred_key_storage_status_period` as small as possible but non-negative. If no such KA is available to the Wallet Unit, it SHALL obtain a new KA from the Wallet Provider that satisfies `key_storage_status.exp` - current time >= `preferred_key_storage_status_period`.

If a `preferred_client_status_period` field is included in the Credential Issuer Metadata, then the Wallet Unit SHALL send the WIA available to them with (`client_status.exp` - current time) - `preferred_client_status_period` as small as possible but non-negative. If no such WIA is available to the Wallet Unit, it SHALL request a new WIA from the Wallet Provider that satisfies `client_status.exp` - current time >= `preferred_client_status_period`.

> Note that the remaining status maintenance period of the KA and WIA impacts the revocation mechanism. A longer status maintenance period will require the revocation status list to be maintained for a longer period of time. A shorter status maintenance period may result in more frequent issuance of KAs/WIAs. It may therefore be beneficial to issue KAs/WIAs with several different status maintenance periods to a single Wallet Unit, such that those with a longer status maintenance period can be used by the Wallet Unit during the issuance of PIDs (for which revocation chaining is mandatory) and those with a shorter status maintenance period may be used for other attestations. As a result of the above requirements a PID Provider or Attestation Provider can always be ensured to receive a KA whose revocation status is maintained for at least 1 month by requesting a KA with a `preferred_key_storage_status_period` of 1 month.

The `key_attestation` JWT contains a field `exp` denoting the technical expiration period of the `key_attestation`.

A Wallet Provider SHALL keep track of which WIAs are associated with which Wallet Units.

In case a Wallet Unit is to be revoked, a Wallet Provider SHALL revoke all `client_status.status` entries associated with that Wallet Unit. 

Regarding revocation of a WSCD or keystore: If the Wallet Provider uses option 1 (type-shared index), the Wallet Provider SHALL only revoke a `key_storage_status.status` entry if the type of WSCD or keystore has a security vulnerability.  Under Option 2 (per-KA index), the `key_storage_status.status` entries associated with the Wallet Unit's KAs can additionally be revoked upon user request (e.g., due to loss or theft of the WSCD or keystore).

During re-issuance of a PID or attestation (e.g., if the technical validity of a PID or attestation expires before the administrative validity period expires), then the Wallet Unit  SHALL send a new KA (i.e., a KA that has not been used before) in the *Credential Request* to the PID Provider or Attestation Provider.

### 2.4.3 PID Provider and Attestation Provider Responsibilities

The technical validity period of a PID SHALL end before the `client_status.exp` of the WIA and the `key_storage_status.exp` of the KA shown to the PID Provider in the issuance process.

Other Attestation Providers (i.e., non-PID Providers) MAY choose a technical validity period of the attestations they issue independently of the `client_status.exp` of the WIA and the `key_storage_status.exp` of the KA received during issuance.

A PID Provider SHALL check the revocation status of **both** the WIA and the KA received during issuance at least once every 24 hours for the validity period of the PID. If either is revoked, the PID Provider SHALL revoke the PID. If PID Providers issue PIDs with a validity period of less than 24 hours, they only need to verify the revocation status of both upon issuance.

> **Note:** In order for PID Providers (and other Attestation Providers) to perform ongoing revocation checks of the WIA, the `client_status` information from the WIA must be made available to the Credential Issuer. Since the WIA is presented to the Authorization Server (as a client attestation) rather than directly to the Credential Issuer, this information must be passed from the Authorization Server to the Credential Issuer by some means. One way to achieve this is to include the relevant `client_status` information in the Access Token issued by the Authorization Server.

As the revocation status list is publicly available, an Attestation Provider can check the revocation status of the WIA and KA and revoke its attestations when either is revoked, if the Attestation Provider wishes to do so. In that case, the Attestation Provider SHALL ensure that the technical validity period of an attestation ends before the `client_status.exp` of the WIA and the `key_storage_status.exp` of the KA shown to the Attestation Provider in the issuance process.

## 2.5 Revocation

Status Lists, as defined in [IETF Token Status List](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/11), SHALL be used as the revocation mechanism for both WIAs and KAs, as described in Appendices D and E of the OID4VCI specification respectively.

To ease scalability, the Wallet Provider can use the following optimisations:

* The status list SHOULD be divided up when Wallet Providers have a sizeable amount of users or issued attestations. The strategy for chunking is left to Wallet Provider discretion. Considerations SHOULD include the size of a status list for downloading and the privacy of the user.
* Multiple status lists MAY be active for a single Wallet Provider.
* The status list SHALL be compressed to reduce the size.

> Note that many chunking strategies exist. It can be based on a fixed size or by time periods, a WP may use a single active status list until it is full or it may have multiple active status list, indices may or may not be re-used, etc. All strategies have pros and cons, which should be evaluated by the Wallet Provider, considering scalability, privacy and interoperability.

### 2.5.1 WIA Revocation and Optional Per-Issuer Status Entry Reuse

The `client_status.status` entry in a WIA represents the revocation state of the Wallet Instance. A Wallet Instance is revoked when the Wallet Provider detects a security vulnerability in the Wallet Instance's device or operating environment, or when the user requests revocation (e.g., due to loss or theft of the device).

For privacy reasons, Wallet Providers SHALL take into consideration the scale of their deployment and the underlying architecture when determining the size of WIA status lists, ensuring that they are sufficiently large to prevent correlation and protect user privacy. At a minimum, a status list SHALL, where appropriate, relate to at least 10000 attestations.

A Wallet Provider MAY assign a single status list entry to all WIAs that a given Wallet Unit presents to the same PID Provider or Attestation Provider (the "per-issuer reuse" option). If this option is used, it applies to the `client_status.status` entry. If this option is used:

* The Wallet Unit SHALL maintain state on which status list entry it has been assigned for each PID Provider or Attestation Provider it has previously interacted with, and SHALL request a WIA referencing that existing entry when interacting with the same PID Provider or Attestation Provider again.
* The Wallet Provider SHALL verify that the Wallet Unit is entitled to reuse the existing status list entry before issuing a new WIA with that entry.
* The Wallet Provider SHALL NOT reuse the same entry for interactions with different PID Providers or Attestation Providers.

If the per-issuer reuse option is used, the Wallet Provider will be able to determine how many PID Providers and Attestation Providers a Wallet Unit has interacted with and how frequently it interacts with each. Wallet Providers SHALL document whether they use this option in their Wallet Provider privacy policy.

If the per-issuer reuse option is not used, the Wallet Provider SHALL assign a fresh, unlinkable status list entry to each WIA it issues.

### 2.5.2 KA Revocation

Depending on the index assignment option chosen by the Wallet Provider (see below), the `key_storage_status.status` entry in a KA represents either the revocation state of the WSCD or keystore *type* used to store the attested keys (Option 1), or the revocation state of the individual Wallet Unit's WSCD or keystore instance (Option 2). Under both options, a KA SHALL be revoked when a security vulnerability is identified that affects the WSCD or keystore -- for example, a certification withdrawal or a cryptographic weakness becomes known in a specific hardware component. Under Option 1, a single revocation action affects all Wallet Units using that type. Under Option 2, revocation affects only the individual Wallet Unit; additionally, the Wallet Provider MAY revoke the entry upon user request (e.g., due to loss or theft of the device).

Under Option 1, the `key_storage_status.status` entry of a KA SHALL NOT be revoked to express revocation of an individual Wallet Instance; individual Wallet Instance revocation SHALL be expressed through the WIA `client_status.status` entry as described in [Section 2.5.1](#251-wia-revocation-and-optional-per-issuer-status-entry-reuse). Under Option 2, the WIA `client_status.status` entry remains the primary means of Wallet Instance revocation, but the `key_storage_status.status` entries associated with that Wallet Unit's KAs MAY also be revoked to reflect that the individual Wallet Unit's WSCD or keystore instance is no longer trusted.

A Wallet Provider SHALL choose one of the following index assignment options for `key_storage_status.status`:

**Option 1: Type-shared index.**
All KAs attesting keys stored in the same WSCD or keystore type SHALL reference the same status list index in `key_storage_status.status`. This means a single revocation action invalidates all KAs of the affected type across all Wallet Units. Because all KAs of the same WSCD or keystore type share a single status list index, the number of entries in a KA status list reflects the number of distinct WSCD or keystore types supported by the Wallet Provider, not the number of deployed Wallet Units. The privacy considerations that motivate the minimum status list size for WIA status lists therefore do not apply to KA status lists given that there are sufficiently many Wallet Units using the same WSCD or keystore type.

**Option 2: Per-KA index with optional per-issuer reuse.**
A Wallet Provider MAY instead assign a fresh status list index to each KA for the `key_storage_status.status` entry (the "per-KA index" option). Under this option, each index represents the revocation state of the specific WSCD or keystore instance as attested in that KA, not the WSCD or keystore type. If this option is used:

* The Wallet Provider MAY further allow a Wallet Unit to reuse the same status list index in all KAs it presents to the same PID Provider or Attestation Provider (the "per-issuer reuse" sub-option). If per-issuer reuse is applied:
  * The Wallet Unit SHALL maintain state on which status list index it has been assigned for each PID Provider or Attestation Provider it has previously interacted with, and SHALL request a KA referencing that existing index when interacting with the same PID Provider or Attestation Provider again.
  * The Wallet Provider SHALL verify that the Wallet Unit is entitled to reuse the existing status list index before issuing a new KA with that index.
  * The Wallet Unit SHALL NOT reuse the same index for interactions with different PID Providers or Attestation Providers. This prevents cross-issuer linkability via the `key_storage_status.status` index.
* If per-issuer reuse is not used, the Wallet Provider SHALL assign a fresh, unlinkable status list index to each KA it issues for a given Wallet Unit.
* For privacy reasons, Wallet Providers using per-KA indices SHALL take into consideration the scale of their deployment and the underlying architecture when determining the size of KA status lists, ensuring that they are sufficiently large to prevent correlation and protect user privacy. At a minimum, a status list SHALL, where appropriate, relate to at least 10000 attestations.
* If the per-issuer reuse sub-option is used, the Wallet Provider will be able to determine how many PID Providers and Attestation Providers a Wallet Unit has interacted with and how frequently it interacts with each. Wallet Providers SHALL document whether they use this sub-option in their Wallet Provider privacy policy.

## 2.6 Signature Algorithms

For signing WIAs, KAs, related proof-of-possessions and Token Status Lists, one of the following algorithms SHALL be used:

* ES256 (ECDSA with SHA-256 and P-256)
* ES384 (ECDSA with SHA-384 and P-384)
* ES512 (ECDSA with SHA-512 and P-521)

No other curve-algorithm combinations are allowed for these purposes.

PID Providers and Attestation Providers SHALL support all of these, whereas Wallet Providers MAY choose to support only one or multiple of these.

# 3 Appendix

## 3.1 Examples

A number of different data objects are sent in the OID4VCI protocol. Below we provide some non-normative examples:

Example of a key-bound *Wallet Instance Attestation*:

```
{
  "typ": "oauth-client-attestation+jwt"
  "alg": "ES256",
  "x5c": ["MIIDDTCCA...]
}
.
{
  "sub": "https://client.example.com",
  "wallet_name": "SmartWallet-mobile",
  "wallet_version": "1.0.1",
  "wallet_link": "https://example.org/wallets/SmartWallet-mobile/info",
  "wallet_solution_certification_information": "https://example.org/certification/SmartWalletMobile/1-0-1/",
  "exp": 1300819380,
  "client_status": {
    "status": {
      "status_list": {
        "idx": 1337,
        "uri": "https://revocation_url/wia-statuslists/42"
      }
    },
    "exp": 1303497780
  },
  "cnf": {
    "jwk": {
      "kty": "EC",
      "use": "sig",
      "crv": "P-256",
      "x": "18wHLeIgW9wVN6VD1Txgpqy2LszYkMf6J8njVAibvhM",
      "y": "-V4dS4UaLMgP_4fY4j8ir7cl1TXlFdAgcx55o7TkcSA"
    }
  }
}
```

> In this example, `exp` indicates the WIA expires in less than 24 hours, while `client_status.exp` is set 31 days after issuance -- the Wallet Provider commits to maintaining the revocation status at index 1337 for at least that long.

Example of how a WIA is transported in a *Pushed Authorization Request* (the WIA is the `OAuth-Client-Attestation` and the Proof-of-Possession of the WIA is the `OAuth-Client-Attestation-PoP`):

```
REQUEST: https://ec.dev.issuer.eudiw.dev/pushed_authorization
METHOD: POST
COMMON HEADERS
-> Accept: application/json; application/json
-> Accept-Charset: UTF-8
-> DPoP: eyJhbGciOiJFUzI1NiIsInR5cCI6ImRwb3Arand0IiwiandrIjp7Imt0eSI6IkVDIiwiY3J2IjoiUC0yNTYiLCJ4IjoiYnhTQlE3RVp0LWw3OF...
-> OAuth-Client-Attestation: eyJ0eXAiOiJvYXV0aC1jbGllbnQtYXR0ZXN0YXRpb24rand0IiwiYWxnIjoiRVMyNTYiLCJ4NWMiOlsiTUlJRERUQ0...
-> OAuth-Client-Attestation-PoP: eyJhbGciOiJFUzI1NiIsInR5cCI6Im9hdXRoLWNsaWVudC1hdHRlc3RhdGlvbi1wb3Arand0In0.eyJpc3MiOi...
CONTENT HEADERS
-> Content-Length: 262
-> Content-Type: application/x-www-form-urlencoded; charset=UTF-8
BODY Content-Type: application/x-www-form-urlencoded; charset=UTF-8
BODY START
client_id=eudiw-abca&response_type=code&redirect_uri=eu.europa.ec.euidi%3A%2F%2Fauthorization&scope=eu.europa.ec.eudi.pid_mdoc&state=XW1M3KRrPtK6y5KfuJPUTqciDoCA_kgbB6NDtIQpx6s&code_challenge=kPeBu-VK_Uu4NfN60Eqv40LxYqjmcQG0v_5TzmU7niI&code_challenge_method=S256
BODY END
```

Example of a *Credential Request* with a `jwt` proof:

```
{
  "credential_configuration_id": "org.iso.18013.5.1.mDL",
  "proofs": {
    "jwt": [
		"eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8xIiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ..."
	]
  }
}
```

Example of a `jwt` object (part of a *Credential Request*):

```
{
  "typ": "openid4vci-proof+jwt",
  "alg": "ES256",
  "key_attestation": "eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8xIiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ..."
}.{
  "aud": "https://credential-issuer.example.com",
  "iat": 1701960444,
  "nonce": "LarRGSbmUPYtRYO6BQ4yn8"
}
```

Example of a `key_attestation` object:

```
{
  "typ": "keyattestation+jwt",
  "alg": "ES256",
  "x5c": ["MIIDQjCCA..."]
}
.
{
  "iat": 1516247022,
  "exp": 1541493724,
  "certification": "https://example.org/certification/wscd/GlobalPlatform/",
  "key_storage_status": {
    "status": {
      "status_list": {
        "idx": 7,
        "uri": "https://revocation_url/wua-type-statuslists/3"
      }
    },
    "exp": 1544172124
  },
  "attested_keys": [
    {
      "kty": "EC",
      "crv": "P-256",
      "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
      "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
    }
  ],
  "key_storage": [
    "iso_18045_high"
  ],
  "user_authentication": [
    "iso_18045_high",
    "iso_18045_moderate"
  ]
}
```

> In this example, `idx: 7` is shared by all KAs for the same WSCD type. `key_storage_status.exp` is set 31 days after the KA's `exp`.

Example of a *Credential Request* with a `attestation` proof:

```
{
  "credential_configuration_id": "org.iso.18013.5.1.mDL",
  "proofs": {
    "attestation": [
		"eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8xIiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ..."
	]
  }
}
```

## 3.2 Mapping to ETSI TS 119 602 v1.1.1

ETSI TS 119 602 v1.1.1 defines the List of Trusted Entities (LoTE), a format used for publishing trusted list infrastructure. The tables below map the fields of the WIA and KA to the corresponding fields in ETSI TS 119 602, to assist implementers working with both specifications.

In the LoTE model, the Wallet Provider corresponds to the **Trusted Entity** (TE), and the Wallet Solution or WSCD/keystore type corresponds to the **Service** offered by that TE.

### 3.2.1 WIA Field Mapping

| WIA field     | ETSI TS 119 602 field | ETSI section | Annex E reference |
|---------------|-----------------------|--------------|-------------------|
| `wallet_name` | `ServiceName`         | 6.6.2        | Table E.3         |

> **Note:** The Wallet Provider identity is inferred from the signing certificate in the `x5c` header parameter. The `TEName` mapping applies to that certificate's subject identity.

### 3.2.2 KA Field Mapping

> **Note:** The Wallet Provider identity is inferred from the signing certificate in the `x5c` header parameter. The `TEName` mapping applies to that certificate's subject identity.
