<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification of Wallet Unit Attestations (WUA) used in issuance of PID and Attestations

## Abstract
The present document specifies how WUA is used in connection with PID Providers and Attestation Providers.

#### [GitHub discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/discussions/450)

## Versioning

| Version | Date       | Description                                                                      |
|---------|------------|----------------------------------------------------------------------------------|
| `0.1`   | 28.03.2025 | Initial version for first discussions.                                           |
| `0.2`   | 14.04.2025 | Improvements after first round of feedback and improved scoping.                 |
| `0.3`   | 28.04.2025 | Addition of Wallet App Attestation, improvements after second round of feedback. |
| `0.4`   | 16.05.2025 | Improvements after commenting. |
| `1.0`   | 20.08.2025 | Changes to WUA Use Cases section (removed references to presentation WUA).       |
| `1.1`   | 05.11.2025 | Editorial changes, rename “Wallet App Attestation (WAA)” to “Wallet Instance Attestation (WIA)”, remove the "ephemeral WIA", and add attestation proof type. |

## 1 Introduction and Overview
The WUA (Wallet Unit Attestation) topic has been discussed in the European Digital Identity Cooperation Group. As a result a number of High Level Requirements (HLRs) have been proposed. The present document is set to enable actors in the EUDIW ecosystem to follow the HLRs while ensuring the interoperability of the ecosystem. The HLRs are available in the [ARF ANNEX 2 Topic 9](https://eu-digital-identity-wallet.github.io/eudi-doc-architecture-and-reference-framework/latest/annexes/annex-2/annex-2-high-level-requirements/#a239-topic-9---wallet-unit-attestation).

This specification is designed to enable the high level requirements defined in the ARF. Additionally the specification strives to ensure that the WUA mechanism is compatible with the existing technical specifications for the EUDIW ecosystem. I.e., the mechanism must be compatible with ISO18013-5 as well as OID4VCI, OID4VP. The goal of the specification, is that the solution is technically simple and does not introduce unnecessary complexity.

[Section 2](#2-solution-description) of this document will serve as the contents of the technical specification and the appendix informally discusses the different solutions that have been proposed during development of this technical specification.

### 1.1 WUA Use Case (from ARF HLRs)
The HLRs of the ARF mandates that the WUA functionality must support a certain set of information to be transferred from the Wallet Provider (via the Wallet Unit) to the Issuing entities (i.e., PID Provider or Attestation Provider), hence we define WUAs as:

* *WUAs*: Wallet Unit Attestations which information will be transferred from a Wallet Provider to the Issuer via the Wallet during issuance. WUAs must allow Issuers to determine the security level of the wallet, authenticate the wallet and check that it has not been revoked for the lifetime of the attestation they are issuing.

### 1.2 Scope
This STS will specify the following:

* The *transfer* of WUAs between the wallet and the issuing party (i.e., either PID or Attestation Provider).
* The *format* of WUAs including its encoding and integrity protection mechanism.
* The *content* of WUAs.
* The *life cycle* of WUAs.
* The *revocation mechanism* for WUAs.

> Note that _how_ Wallet Providers issue WUAs to the wallet are out of scope for this technical specification, as this will only be done by the wallet providers themselves and does therefore not require any standard to achieve interoperability.

## 1.3 Requirements Notation
The key words "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119) [RFC8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they are written in all capital letters.

# 2 Solution Description
EUDI Wallets shall use two types of attestations in relation to issuance: 1) A *Wallet Instance Attestation* (WIA) that attests *only* the integrity of the app and 2) a *Wallet Unit Attestation* (WUA) that attests the security of keys in the WSCD and that the Wallet Unit has not been revoked.

The WIA allows PID and Attestation Providers to protect their endpoints by only communicating with Wallet Applications which integrity are ensured by the Wallet Providers. WUAs on the other hand allow PID and Attestation Providers to ensure that they only issue attestations that are cryptographically bound to keys that are properly protected (i.e., in a WSCD with sufficiently high attack resistance), as well as to revoke their credentials in case a Wallet Provider revokes a Wallet Unit.

A high level overview of the current solution is given in the table below:

| **Conceptual Part** | **Solution** |
|---------------------|----------------------------------------------------------------------------------|
| Format              | Both WIA and WUA shall be JSON Web Tokens (JWTs) signed by the Wallet Provider. See [Section 2.1 Format](#21-format).   |
| Transport           | The WIA shall be a client attestation sent to token endpoint at Authorization Server.  The WUA shall be a `key_attestation` element sent within the `CredentialRequest` of OID4VCI (either as `attestation` proof type or in the header of a `jwt` proof type). The `key_attestation` element is extended to include a `eudi_wallet_info` element, containing EUDI Wallet specific information.  See [Section 2.2 Transport](#22-transport).   |
| Content             | The WIA shall contain only very basic information about the application. Certain WUA information will be transferred in existing elements of the OID4VCI protocol, e.g., revocation information and the public key corresponding to a private key stored in the WSCA/WSCD. Other WIA and WUA information will be a contained in a `eudi_wallet_info` element. The concrete contents are discussed in [Section 2.3 Content](#23-content). |
| Life Cycle          | The WIA shall be have a very short time-to-live. The WUA shall be issued by Wallet Providers that may issue WUAs with different validity periods to the same. They must however maintain the revocation status for the entire validity period of the WUA if longer than 24 hours. See [Section 2.4 Life Cycle](#24-life-cycle).    |
| Revocation          | WIA shall not, due to their short time-to-live, have any revocation mechanism associated. Revocation of WUAs shall be done using chunked Status List. See [Section 2.5 Revocation](#25-revocation).|

Below we present details of the technical specification for WUAs.

## 2.1 Format
A Wallet Instance Attestation SHALL be a JSON Web Token (JWT) as specified in [RFC7519](https://datatracker.ietf.org/doc/html/rfc7519).

A Wallet Unit Attestation SHALL be a JSON Web Token (JWT) as specified in [RFC7519](https://datatracker.ietf.org/doc/html/rfc7519).

## 2.2 Transport
The ARF specifies that [OIDF OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3) is to be  used for issuance, hence the transport of the WIA and WUA must be compatible with the options of the OID4VCI protocol. Note that OID4VCI uses `Credential Issuer Metadata` and `Authorization Server Metadata` to specify the need for WIA and WUA.

### 2.2.1 Transport of WIA
The WIA SHALL be an Wallet Attestation as specified in [Appendix E of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3) and sent to the Authorization Server in the Pushed Authorization Request and the Token Request.

The WIA SHALL be signed by the Wallet Provider.

The WIA SHALL  be sent along with a Proof-of-Possession (PoP), as specified in [Appendix E of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3).

> Note that the WIA is a mechanism which PID and Attestation Providers can use to protect their endpoints.

When a WIA is sent to the Wallet, it SHALL have a time-to-live of less than 24 hours. I.e., the difference between expiration time `exp` and the time of issuance SHALL be less than 24 hours.


#### 2.2.1.1 Wallet Providers Responsibilities for Transport of WIAs
A Wallet Provider SHALL verify the integrity of the Wallet Application before signing a WIA.

The Wallet Provider SHALL ensure that a Wallet Unit has WIAs as needed for issuance.

Wallet Provider SHALL ensure that their Wallets only use a WIA once.

> This is to prevent issuer linkability.

#### 2.2.1.2 PID Providers and Attestation Providers Responsibilities for Transport of WIAs
When a PID Provider or Attestation Provider receives a WIA, then they SHALL check that the signature of the JWT verifies under the Wallet Provider's public key as found on the Trusted List of Wallet Providers.

When a PID Provider or Attestation Provider receives a WIA, then they SHALL check that it has not expired.

When a PID Provider or Attestation Provider receives a WIA, then they SHALL check the signature of the PoP verifies under the present in the `cnf`.

### 2.2.2 Transport of WUA
The WUA SHALL be a `key_attestation` as defined in [Appendix D of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3) but extended with a  `eudi_wallet_info` as presented in [Section 2.3 Content](#23-content).

During issuance in OID4VCI, EUDI Wallets SHALL in the `proofs` field include the WUA in a proof of type `jwt` or `attestation` with their *CredentialRequest* to the Credential Issuer.

> Note the `jwt` acts as a proof-of-possesion (PoP) of the keys.

#### 2.2.2.1 Wallet Providers Responsibilities for Transport of WUAs
The WUA (i.e., the `key_attestation` element) SHALL be generated and signed by the *Wallet Provider*.

The Wallet Provider SHALL verify that the keys attested to in the `key_attestation` are stored in secure hardware as described in the `key_attestation`.

The `attested_keys` element of an `key_attestation` SHALL contain at least one key, but MAY contain multiple keys (in order to support batch issuance).
If a Wallet include their WUA in a `jwt` element, then it SHALL be signed by the *Wallet Unit* with the key at index 0 of the `attested_keys` array within the `key_attestation` object.

> Requiring only one signature on the entire `jwt` may improve the user experience as some WSCDs may require a user gesture to sign. Note that this does not degrade the security as the signature of the Wallet Provider still binds all all the keys to the same WSCD.

Wallet Providers SHALL ensure that their Wallets use a WUA only once and each individual key SHALL only be included in one WUA.

> This is to prevent verifier linkability. Note that the attested keys in a WUA cannot be re-issued.

#### 2.2.2.2 PID Providers and Attestation Providers Responsibilities for Transport of WUAs
PID and Attestation Providers SHALL verify that the signature of the WUA verifies under the Wallet Provider's public key as found on the Trusted List for Wallet Providers.

PID and Attestation Providers SHALL verify that the signature of the `jwt` element matches the key at index 0 of the `attested_keys` array within the `key_attestation` object included in the `jwt`.

If a PID or Attestation Providers receives a WUA in a `jwt` proof type, then they SHALL verify that the `jwt` element includes a valid `c_nonce` from their `nonce_endpoint` in the `nonce` field of the `jwt`.

## 2.3 Content
The high-level requirements of the ARF require a number of different attributes being transferred as part of the WIA and WUA. Some of these attributes are already defined by the OID4VCI specification, in which case OIDC4VCI will be used. In particular, the `user_authentication` and `secure_storage` attributes of the `key_attestation` SHALL be used to indicate the 'attack potential resistance' of the WSCD where the attested keys are stored.  Other attributes are specific for the EUDI Wallet ecosystem, these are placed in a `eudi_wallet_info` element in both the WIA and WUA.

> As the content of the `eudi_wallet_info` is related to the content of the Trusted List, this content specification must be updated when the content of the Trusted List has been defined. In particular, the information in `general_info` must be aligned with the Trusted List, such that the information is uniquely linked to an entry on the Trusted List.

The `eudi_wallet_info` object contains the following informational content:

| Attribute | Multiplicity | Type | Description |
|---------------|--------------|-----------------------------------|--------|
| `general_info` | REQUIRED in both WIA and WUA| [`general_info`](#231-generalinfo) | specifies generic information on the Wallet Unit. |
| `wscd_info` | REQUIRED in WUA | [`wscd_info`](#232-wscdinfo) | specifies information on the WSCD containing the attested keys. |

### 2.3.1 `general_info`
The `general_info` object has the following content:

| Attribute | Multiplicity | Type | Description |
|---------------|--------------|-----------------------------------|--------|
| `wallet_provider_name` | REQUIRED | *string* | **Name of Wallet Provider**, as listed on the Trusted List of Wallet Providers |
| `wallet_solution_id` | REQUIRED | *string* | **Identifier of the Wallet Solution**, as listed on the Trusted List of Wallet Providers. |
| `wallet_solution_version` | REQUIRED | *string* | **Version of the Wallet Solution** |
| `wallet_solution_certification_information` | REQUIRED | *JSON/string* | Object containing information on which conformity assessment body certified the Wallet Solution, the applicable certification number, etc.  |

> As the certification scheme has not yet been defined, the exact content of `wallet_solution_certification_information` is undefined. This content will be defined in a future update. Note that the fields are subject to match the specification for Trusted Lists once it is released.

### 2.3.2 `wscd_info`
The `wscd_info` object has the following content:

| Attribute | Multiplicity | Type | Description |
|---------------|--------------|-----------------------------------|--------|
| `wscd_type` | RECOMMENDED | *string* | Technical implementation of the WSCD, one of the following values: "REMOTE", "LOCAL_EXTERNAL", "LOCAL_INTERNAL", "LOCAL_NATIVE" or "HYBRID" as described in the ARF. |
| `wscd_certification_information` | REQUIRED | *JSON / string* | Information about the certification achieved by the WSCD, such as under which scheme (for instance, Common Criteria, GlobalPlatform), the requirements that were evaluated (for example, the Protection Profile used), the evaluation level, and perhaps other applicable information. |

> As the certification scheme has not yet been defined, the exact content of `wscd_certification_information` is undefined. This content will be defined in a future update. Note that the OID4VCI specification does specify a `certification` attribute in the `key_attestation` element, that could be used instead of `wscd_ certification_information`, if the certification data is to be made available on a URI.


### 2.3.3 Content of WIA
The content of the WIA is given by [Appendix E of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3). In addition to the required attributes from the specification, the WIA SHALL also include the `eudi_wallet_info` claim, containing `general_info` described in [*general_info*](#231-generalinfo). 

### 2.3.4 Content of WUA
The content of the WUA is given by the `key_attestation` definition specified in [Appendix D of OpenID for Verifiable Credential Issuance v1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3). If a WUA is sent as an attestation proof type, then it SHALL also include a `c_nonce` as specified in [Appendix F.3 of OpenID for Verifiable Credential Issuance v. 1.0](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0-final.html#appendix-F.3).
In addition to the required attributes, the WUA SHALL also include the `eudi_wallet_info` claim, containing both `general_info` described in [*general_info*](#231-generalinfo) and `wscd_info` described in [*wscd_info*](#232-wscdinfo). The WUA SHALL also include the `status` object.

> OID4VCI natively supports revocation, hence the `status` object of the `key_attestation` element SHALL be used to handle revocation. In particular, the value of `idx`  is what allows the Wallet Provider to identify the Wallet Unit (i.e. the revocation identifier) and is therefore what is required by the PID provider to request a revocation of the Wallet Unit.

> Note that the `key_attestation` element contains the signed keys in `jwk` format. The `jwk` format specifies that each key must contain an algorithm type. Hence, this provides protection against downgrade attacks for attested keys.

> Also note that the `key_attestation` element contains an `iss` element, i.e. the issuer identifier, which also includes the Wallet Provider identity.

## 2.4 Life Cycle
WIAs are short-lived and to be consumed upon usage, hence this section only specifies the life cycle of the WUAs.

### 2.4.1 Wallet Provider responsibilities
A Wallet Provider SHALL choose the technical validity period of the WUA and SHALL maintain the revocation mechanism i.e., see [Section 2.5](#25-revocation) for until this period has passed.

A Wallet Provider MAY choose to issue WUAs with different validity periods. When choosing the validity period, the Wallet Provider SHALL at least consider security, user privacy and interoperability, and the needs of the PID Provider.

A Wallet Provider SHALL ensure that a Wallet can always present a with a validity period of at least 1 month.

A Wallet Unit SHALL send a WUA with a validity of at least one month to PID Providers during PID issuance.

> Note that the validity period has impact on the revocation mechanism. A longer validity period will require the revocation status list to be maintained for a longer period of time. A shorter validity period may result in more frequent issuance of WUAs. It may therefore be beneficial to issue WUAs with several different validity periods to a single wallet such that those with a longer validity period can be used by the Wallet Unit during the issuance of PID where revocation chaining is mandatory and those with a shorter validity period may be used for other attestations.

The `key_attestation` JWT contains a field `exp` denoting the technical expiration period of the `key_attestation`.

A Wallet Provider SHALL keep track of which WUAs are associated with which Wallet Units.

In case a Wallet Unit is to be revoked, a Wallet Provider SHALL revoke all WUAs associated with this Wallet Unit.

During re-issuance of a PID or Attestation (i.e., if for example the technical validity of a PID or Attestation expires before the administrative validity period expires), then the Wallet Unit (or Attestation Provider) SHALL send a new WUA (i.e., a WUA that has not been used in relation to issuance before) in the *Credential Request* to the PID or Attestation Provider.

### 2.4.2 PID Provider and Attestation Provider responsibilities
The technical validity period of a PID or Attestation SHALL end before the technical validity period of the WUA shown to the PID in the issuance process.

Other Attestation Providers MAY choose a technical validity period of the attestations they issue independently of the technical validity for the WUA used during issuance.

A PID Provider SHALL check the revocation status of the WUA received in relation to issuance at least once every 24 hours for the validity period of the PID. If the WUA is revoked, then the PID Provider SHALL revoke the PID. If PID Providers issue PIDs with a validity period of less than 24 hours, they only need to verify the validity period of the WUA upon issuance.

As the revocation status list is publicly available, the Attestation Providers MAY check the revocation status of the WUA and revoke their attestations if the WUA is revoked, if the Attestation Provider wishes to do so.

> Note it is required to be a new WUA as the keys should be different from previous WUA in order to prevent linkability of keys upon presentation.

## 2.5 Revocation
If a WUA has a validity period longer than 24 hours, then Status lists, as defined in [IETF Token Status List](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/11) SHALL be used as revocation mechanism as described in Appendix D and E of the OID4VCI specification for WUAs, i.e. the `status` element of `key_attestation` SHALL be used.

To ease scalability, the Wallet Provider can use the following optimisations:
* The status list SHOULD be divided up when issuers have a sizeable amount of users or issued attestations. The strategy for chunking is left to issuer discretion. Considerations include size of a status list for downloading and privacy of the user. For privacy reasons, a status list SHOULD relate to at least 10000 attestations.
* Multiple status lists MAY be active for a single Wallet Provider.
* The status list SHALL be compressed to reduce the size.

> Note that many chunking strategies exist. It can be based on a fixed size or by time periods, a WP may use a single active status list until it is full or it may have multiple active status list, indices may or may not be re-used, etc. All strategies have pros and cons, which should be evaluated by the Wallet Provider, considering scalability, privacy and interoperability.

# 3 Appendix

## 3.1 Example JSON

A number of different data objects are sent in the OID4VCI protocol. Below we provide some non-normative examples:

Example of a key bound *Wallet Instance Attestation*:
```
{
  "typ": "oauth-client-attestation+jwt"
  "alg": "ES256",
  "kid": "11"
}
.
{
  "iss": "https://client.example.com",
  "sub": "https://client.example.com",
  "eudi_wallet_info": {JSON},
  "exp": 1300819380,
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

Example of a *Credential Request*:
```
{
  "credential_configuration_id": "org.iso.18013.5.1.mDL",
  "proofs": {
    "proof_type": "jwt",
    "jwt": "eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8xIiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ"
  }
}
```

Example of a `jwt` object (part of a *Credential Request*):
```
{
  "typ": "openid4vci-proof+jwt",
  "alg": "ES256",
  "kid": "0",
  "key_attestation":["eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8xIiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ"] 
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
  "iss": "<identifier of the issuer of this key attestation - Corresponds to `wallet_provider_name` in `eudi_wallet_info`>",
  "iat": 1516247022, 
  "exp": 1541493724,
  "eudi_wallet_info": {JSON},
  "status": { 
    "status_list": {
        "idx": 412,
        "uri": "https://revocation_url/statuslists/1"
      }
  }
  "attested_keys": [
    {
      "kty": "EC",
      "crv": "P-256",
      "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
      "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
    }
  ]
}
```

Example of the `eudi_wallet_info` object:
```
"eudi_wallet_info": {
  "general_info": {
     "wallet_provider_name": "WalletCorp", 
     "wallet_solution_id": "SmartWallet-mobile",
     "wallet_solution_version": "1.0.1",
     "wallet_solution_certification_information": "https://example.org/certification/SmartWalletMobile/1-0-1/"
  },
  "wscd_info": {
     "wscd_type" : "REMOTE HSM",	
     "wscd_certification_information" : "GlobalPlatform",
     "wscd_attack_resistance" : 2
  }
}
```

## 3.2 Revocation discussion

In this section, we list and discuss possible revocation methods for WUAs that have been investigated during the writing of this STS. The discussion revolve around possible revocation mechanisms for revocation of WUAs in relation to the revocation chaining requirement for PID Providers. Note that the discussion only focuses on WUAs used during issuance of PIDs and other attestations.

We will use the following abbreviations when discussing the solutions.

*n*: number of WP users.
*n_iss*: number of Wallet Units using the issuer.
*m*: Avg. number of issuers of PID or Attestations used by each Wallet Unit.

| Name of method | Description        | Pro  |  Cons                        |
|---------|-------------|----------------------------------|---------------------------|
| Status list  | Each attestation contains a random (unique pr. issuer) revocation identifier (the *status index*) generated and stored by the WP. When the WP revokes a Wallet Unit, it will flip the *status index* bit of the public token status list, i.e. if *status index* bit of the token status list i '1', the attestations with revocation identifier *status index* has been revoked. | Conceptually easy to understand and generally well supported. The WP does not learn which wallet units access which issuers, only that a issuer has accessed the status list. The status list is fairly static and can be published using a CDN or similar technologies. | The WP needs to maintain the status for *n x m* identifiers. Relevant parties need to keep track of both the status index and URI of the status list. |
| Chunked status list | Instead of publishing all revocations on a single status list, the WP may opt to publish the revocations of a subset of the attestations on separate lists. This chunking strategy as left for the WP to decide, but should ensure that the status list is sufficiently large to protect the privacy, i.e. it should contain atleast 10000 entries. | Relevant parties need to keep track of both the status index and URI of the status list. |
| Secret seed PRNG status list | The WP stores a secret seed for each Wallet Unit. This seed is used to generate the revocation identifier. When the WP revokes a Wallet Unit, it will add the secret seed to the public revocation list, allowing the issuers to compute the revocation identifiers. Otherwise this functions as a status list. | The WP will only need to maintain the status for *n* seeds. | After revocation, the seed is published, making it possible to compute and thereby link the revocation identifiers.|  
| Online status protocol | Instead of the issuer downloading a complete status list, the issuer queries the WP for the status of each individual Wallet when doing a revocation check. Note that the Wallet identifier used in this approach can be generated by hashing the issuer identifier and a secret key shared between the Wallet Unit and Wallet Provider. | The WP only needs to maintain the status for the *n* Wallet Units. The Issuer only needs to process the relevant *n_iss* Wallet Units.| Each request will require the WP to do a lookup and cannot be distributed as static data. The WP also learns which issuers have been accessed by the Wallet Unit.|
| Wallet Type lookups | The 'Status list' and 'Online status protocol' methods can be modified to operate on "Wallet types" rather than individual wallets. Instead of a revocation id, the WUA will contain a "Wallet type" attribute, which can be used to revoke "classes" of Wallet Units.| As there will be a very small (compared to individual Wallet Units) number of wallet types, computational complexity and privacy issues are minimal. | The approach does not allow for revocation of individual Wallet Units, which is required for PID use cases. |
| Short lived attestations | As an alternative to revocation, the WUA can expire less than 24 hours after issuance, this would eliminate the need for revocation altogether. | WP does not need to maintain revocation status. | WP will often have to perform verifications of the integrity of the Wallet Unit. Short lived attestations will not accommodate the legal requirement for PID providers to check the revocation status of the Wallet Unit, but could be relevant for other, non-PID, types of attestations. |

#### 3.2.1 Revocation proposal

With the above solutions in mind, we propose to use the chunked status list, e.g. the WP may create a status list of fixed size *x* and use that status list for the next batch of *x* issued WUAs. Once *x* WUAs have been issued, the WP will create a new status list and use that for the next batch of WUAs, etc. The URI of the status list will have a format similar to: `https://revocation_url/statuslists/batch_id/` for WUAs issued in batch *batch_id*. We note that the WP may use  multiple active status lists. We also note that if the *batch_id* is based on a timestamp or counter, it may leak information on the issuance date of the WUA and should be randomly generated.

We note that status lists encourage compression, which will reduce the size in general. A similar approach to revocation status list are used elsewhere in the ARF and it would be preferable if the same method could also be used for WUA. We note that revocation status lists are also natively supported by the *key_attestation* element in OID4VCI.

Regarding the scalability of the described proposal, a rough calculation of the complexity for a status list corresponding to one day is provided below:

The example is based on a Wallet Provider servicing 80 mio. Wallet Units, each having 10 WUAs issued per day, this will result in revocation information for one day for 8 * 10^8 WUAs.

> Note that each wallet having 10 WUAs issued each day probably is an exaggerated estimate by a factor of 5-10. The estimate can be further broken down to *avg. number of issuers for each wallet / lifetime of the WUA*, as it depends on the number of PID providers and Attestations providers each Wallet Unit interacts with and with the lifetime of the WUA: Reducing the lifetime of the WUAs will limit the lifetime of the issued PIDs (or other attestations limited by the lifetime of the WUA) and thereby force more frequent re-issuance. As each re-issuance will also require a new WUA, this requires the Wallet Unit to more frequently request issuance of WUAs.

The total (uncompressed) revocation related information of a single day will therefore be around 8 * 10^8 bits or 100 MB. However, very high compression rates are expected due to the nature of the status lists, and the compressed lists are estimated to be around 1 MB if not smaller.

> Note that in practice, multiple status list would most likely be used in order to reduce the size of each list. For example, if each list contains 10^5 entries, it will give a compressed bit string status list in the range of hundreds of bytes.

The Wallet Provider will need to keep track of which revocation identifier (index of the status list) belongs to which Wallet Unit. Assuming 64 bits are allocated for the revocation identifier, the Wallet Provider will need to store revocation information will require roughly 6.4 GB of storage.
