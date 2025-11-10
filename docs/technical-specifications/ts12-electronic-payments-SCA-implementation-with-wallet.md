
<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification of Strong Customer Authentication (SCA) Implementation with the Wallet

## Abstract

The present document specifies structures and processing rules for conducting Strong Customer Authentication (SCA) according to [PSD2] with use of the European Digital Identity Wallets.

### [GitHub discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/discussions/439)

## Versioning

| Version | Date        | Description                              |
|---------|-------------|------------------------------------------|
| `0.1`   | 10.10.2025  | Initial version for the first discussion |
| `0.5`   | 26.10.2025  | Update after the Focus Meeting 1 and comments from GitHub |
| `0.7`   | 3.11.2025  | Update after the Focus Meeting 2 |
| `0.8`   | 4.11.2025  | Adding the minimum transactional data and attestation metadata specifications |
| `0.9`   | 7.11.2025  | Pre-release version |
| `0.95`   | 10.11.2025  | For review |



## 1 Introduction

### 1.1 Overview

Strong Customer Authentication (SCA) for payments is mandated under [PSD2] Article 97 (1) for electronic payment initiation, as well as for accessing a payment account or other User's actions that present a risk of fraud. By design, the EUDI Wallet supports all three authentication factor categories required by [PSD2] (knowledge, possession, inherence) to enable SCA, as well as Dynamic Linking.

To ensure interoperability, a set of requirements is specified for the implementation of SCA using the Wallet. This technical specification is based on the discussion within Topics W and AA.

### 1.2 Scope of the Document

The [ARF] Annex 2 Topic 20 introduces the concept of dedicated SUA Attestations issued into the user’s wallet by their ASPSP and then later presented in conjunction with transactional data to be signed by the wallet (for Dynamic Linking).

In general, this specification aims to define the behaviour of the Wallet Units when receiving a presentation request of attributes from a SUA Attestation used for SCA (hereinafter called "SCA Attestation"). Therefore, the Wallet Solution Providers shall implement their Wallet Solutions in compliance with this document.

At the same time, the related SCA Attestation Rulebooks, delivered by the financial sector, provides all the necessary information and requirements for:

+ Attestation Providers (Payment Service Providers, such as banks) to issue their SCA Attestations,
+ Relying Parties (Payment Service Providers or other payment actors, such as payees or Payment Initiation Service Providers) to make  presentation requests of the SCA Attestation, as payment transaction authentication.
Therefore the Attestation Providers and Relying Parties shall also follow a related SCA Attestation Rulebook.

This document contains:

- overview of the SCA scenarios at stake,
- minimum requirements and specifications for transactional data send in a presentation request,
- minimum requirements and specifications for metadata of the SCA Attestation,
- transactional data processing and display message rendering requirements,
- other necessary requirements related to Wallet Solutions functionality, interoperability or security.

This document is complemented by a related SCA Attestation Rulebook that may specify:

- SCA Attestation types, applicable in various contexts,
- full/extended specifications of attributes and metadata of the SCA Attestations,
- full/extended specifications of transactional data types transported by the presentation protocol,
- other requirements such as policies for SCA Attestation Providers or Relying Parties.

This document provides the minimum necessary requirements and specifications for the basic SCA flow, to be supported by all the Wallet Solutions. Therefore, the SCA Attestation Rulebooks shall be created in conformity with this specification.
The SCA Attestation Rulebooks may provide additional requirements and specifications extending this document (for instance to improve user experience or customise the scenarios), that may be implemented by Wallet Solutions on the voluntary basis.  

This version of the document focuses on [SD-JWT-VC] format and [OID4VP] presentation protocol only.


### 1.3 Requirements Notation

The key words "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC2119 and RFC8174 when, and only when, they are written in all capital letters.

### 1.4 Terminology

| Term               | Definition   | Based on  |
|--------------------|--------------|-----------|
| Payment Service Provider (PSP) | A credit institution, electronic money institution, post office giro institution, or payment institution offering payment services to Payers and/or Payees. | [PSD2] Article 4 (11) |
| Account Servicing Payment Service Provider (ASPSP); synonyms: Issuer, Payer's PSP, Issuing PSP |Payment Service Provider providing and maintaining a payment account for a Payer and issuing the Payment Credential into his/her Wallet. | [PSD2] Article 4 (17) |
| Account Information Service Provider (AIPSP); synonyms: Open Banking Provider | Payment Service Provider providing an online service to provide consolidated information on one or more payment accounts held by the payment service user with either another payment service provider or with more than one payment service provider | [PSD2] Article (16) and (19) |
| EU Digital Identity Wallet (EUDIW); synonym: Wallet | An electronic identification means which allows the User to securely store, manage and validate person identification data and electronic attestations of attributes for the purpose of providing them to relying parties and other Users of European Digital Identity Wallets. | [eIDAS] Article 1 (3) (j) (42) |
| Payee | A natural or legal person who is the intended recipient of funds which have been the subject of a payment transaction. | [PSD2] Article 4 (9) |
| Payer; synonyms: User, Holder, Account Holder | A natural person who holds a payment account and allows a payment order from that payment account, and who uses an EUDIW provided in accordance with [eIDAS]. | [PSD2] Article 4 (8), [eIDAS] Article 1 (3) (b) |
| Payment Credential; synonym: SCA Attestation | A digital credential in the form of an Electronic Attestation of Attributes, issued by the Attestation Provider into the User's Wallet to facilitate payment use cases. |  |
| Acquirer; synonym: Payee's PSP | The Payment Service Provider of the Payee. | [PSD2] Articles 4 (9) and (11) |
| Merchant | A type of Payee, typically offering goods or services to Users. |  |
| Intermediary | An entity acting on behalf of the Payer's PSP or the Payee. | [eIDAS] Article 1 (5) (5b) (10)  |

Further capitalised words and terms, not included in the above table, are to be interpreted in line with [ARF] Annex 1.

## 2 High-level Solution Design

Note: This chapter is **informative**. It aims to help the reader to understand the context for this specification. All the possible scenarios and flows supported by this specification may be described in a related SCA Attestation Rulebook.  

### 2.1 Process Flows

On a high level, the solution is composed of the following parts:

+ **Registration**: The secure association of the Wallet Unit and Holder with the Account Holder by means of the Payer's PSP issuing a dedicated, cryptographically bound SCA Attestation into the User's Wallet. This step is not described in this document.
+ **Authentication**: The Strong Customer Authentication at time of transaction by means of presenting the Payment Credential.
+ **Dynamic Linking**: Linking of the User's consent to a specific transaction by means of including transaction-specific information in the presentation request and (signed) response.

Using an EUDI Wallet for SCA in general can happen in one of two operating setups:

+ In the first setup, the Wallet Unit is presenting the SCA Attestation back to the Attestation Provider, usually the User's PSP (or an Intermediary on their behalf). The User may be entirely in the context of their PSP (e.g., interacting with their online/mobile banking to send a credit transfer) or he/she may have been redirected from a third party (e.g., a Merchant or a TPP) to the PSP's environment. When used like this, the EUDIW resembles banks' existing authenticator solutions. We refer to this setup as the "**Issuer-requested flow**".
+ In the second setup, the Wallet Unit is presenting the SCA Attestation to a third party which is not the User's PSP (or an Intermediary on their behalf). We refer to this setup as the "**Third-party-requested flow**". It is driven by two constellations:
  + As per [PSD2], the User may give consent to execute a payment not only directly towards their PSP, but also via the Payee (or an Intermediary on their behalf). When used like this, the EUDIW loosely resembles an xPay wallet functionality.
  + The integration models for TPPs such as PISPs and AISPs know a variant called "embedded SCA flow" where the User provides authentication information via the TPP to the ASPSP.

In a Third-party-requested flow, additional communication needs to take place between the third party and the ASPSP to execute the action. This usually happens over existing payment or Open Banking networks and is beyond the scope of this document.

It is expected that SCA Attestation Providers utilise different SCA Attestation types for the Issuer-requested vs. Third-party-requested flows to benefit from the more stringent handling of Embedded Disclosure Policies as defined in this specification for relevant SCA applications in the first setup. SCA Attestation Providers may further provide different SCA Attestation types to represent (a) the User or (b) a specific account or card belonging to a given User with that attestation, according to their needs.

### 2.2 SUA Attestations for SCA

An SUA Attestation for SCA purposes (hereinafter called "SCA Attestation") is an attestation issued by a PSP into a Wallet Unit that serves as a verifiable digital proof that the Wallet Unit is associated with a specific Payer and, if applicable, their account or card managed by the PSP.

In the financial services ecosystem, data structures differ between payment methods. Therefore, there will be multiple types of SCA Attestations. The structures of the SCA Attestations to be used in conjunction with this specification are to be specified in related SCA Attestation Rulebooks.

The following is a minimal, non-normative example of an SCA Attestation that represents a card belonging to a given User:

```json
{
    "iss": "https://issuer.superbank1.com",
    "aud": "https://rp.superbank1.com",
    "sub": "4f473c31-0e0a-4e54-8bb0-b69dc2a03b23",
    "iat": 1683000000,
    "nbf": 1683000000,
    "exp": 1883000000,
    "vct": "https://credentials.cardpaymentauthority.com/card",
    "cnf": {
        "jwk": {
            "kty": "EC",
            "crv": "P-256",
            "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
            "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
        }
    },
    "pan_last_four": "1234",
    "scheme": "Scheme",
    "scheme_logo": "https://www.scheme.com/logo.svg" 
}
```

The following is a minimal, non-normative example of an SCA Attestation that represents an account belonging to a given User:

```json
{
    "iss": "https://issuer.superbank2.com",
    "aud": "https://rp.superbank2.com",
    "sub": "d9c28888-b166-4cf4-bb75-0d374b6214d4",
    "iat": 1683000000,
    "nbf": 1683000000,
    "exp": 1883000000,
    "vct": "https://psd2.standard.org/payment_account",
    "cnf": {
        "jwk": {
            "kty": "EC",
            "crv": "P-256",
            "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemA",
            "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZZ"
        }
    },
    "iban": "DE99370501981234567890",
    "bic": "COLSDE33XXX",
    "currency": "EUR"
}
```

The following is a minimal, non-normative example of an SCA Attestation that just represents a User:

```json
{
    "iss": "https://issuer.superbank3.com",
    "aud": "https://rp.superbank3.com",
    "sub": "37774a3f-ab14-43c3-96bc-bb1066a30a1d",
    "iat": 1683000000,
    "nbf": 1683000000,
    "exp": 1883000000,
    "vct": "https://psd2.standard.org/payment_service_user",
    "cnf": {
        "jwk": {
            "kty": "EC",
            "crv": "P-256",
            "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemA",
            "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZZ"
        }
    }
}
```

### 2.3 SCA Attestation Metadata

A presentation of an SCA Attestation can be requested in conjunction with different transaction types, e.g., logging in to your online banking, or confirming a payment. To prevent fraudulent abuse, Attestation Providers must be able to exercise control over which combinations of transaction types and SCA Attestation types (e.g., for the Issuer-requested vs. Third-party-requested flow variants) are permitted. To do so, this specification extends the standard Verifiable Credential (VC) type metadata functionality of [SD-JWT-VC].

This is achieved by introducing a `transaction_data_types` parameter to the VC type metadata. This parameter contains one entry for each permitted transaction data type for the given SCA Attestation type. Each entry in turn contains a map of key-value pairs that define how to process different aspects of the transaction type in the context of presenting an SCA Attestation.

These aspects comprise the following:

+ a JSON schema defining the structure of the `payload` object within the `transaction_data` object of a given type,
+ a JSON document containing a catalogue of string identifiers and their localised values for the titles and/or descriptions of each field in the schema, for the Wallet Unit to display these localised values on the transaction confirmation screen,
+ a JSON document containing a catalogue of string identifiers and their respective hierarchy level of importance, for the Wallet Unit to visually represent them accordingly on the transaction confirmation screen,
+ a JSON document containing a catalogue of string identifiers and their localized values for UI elements, such as buttons, for the Wallet Unit to use them on the transaction confirmation screen.

Attestation providers are expected to use collision-resistant identifiers for transaction data types. How they are communicated towards the party assembling the `transaction_data` object, which may be the RP, is beyond the scope of this document. It is expected that sector-specific standardisation bodies or schemes will provide this governance and/or automated services for RPs.

Values for the attributes under an identifier are URIs. These can be non-resolvable URIs, in which case the Wallet Solution knows them through this or another specification it adheres to, or they can be resolvable URLs. In this case, Wallet Units will resolve the address and obtain and process the JSON document from the given location.

The following is a non-normative example of an SCA Attestation type metadata:

```json
{
    "vct": "https://pay.example.com/card",
    "name": "pay.example Payment Credential",
    "description": "This is a pay.example Payment Credential",
    "extends": "https://pay.example.com/basic-schema-1.2",
    "extends#integrity": "sha256-9cLlJNXN-TsMk-..5ca_xGgX3c1VLmXfh-WRL5",
    "schema_uri": "https://pay.example.com/card-schema-1.0",
    "schema_uri#integrity": "sha256-o984vn819a48..5t0WRL5ca_xGgX3c1VLmXfh",
    "transaction_data_types": {
        "urn:eudi:sca:payment_authentication:1": {
            "schema": "urn:eudi:sca:payment_authentication:1",
            "i18n": "urn:eudi:sca:payment_authentication:1",
            "visualisation": "urn:eudi:sca:payment_authentication:1",
            "ui": "urn:eudi:sca:payment_authentication:1"
        }
    }
}
```

The detailed specification for these parameters is located in Section 4. An SCA Attestation Rulebook MAY specify additional parameters for optional support by Wallet Solutions.

The SCA Attestation Provider shall publish the metadata parameters in accordance with the specification provided in [SD-JWT-VC].

### 2.4 Standardised Transaction Data

To enable interoperability, this specification extends the `transaction_data` object defined in [OID4VP] by adding a `payload` parameter. This parameter is an object with the transaction details, and its structure is defined by the JSON schema referenced in the `schema` parameter of the SCA Attestation type metadata.

The following is a non-normative example of the transaction data object including the `payload` object:

```json
{
    "type": "urn:eudi:sca:payment_authentication:1",
    "credential_ids": ["payment_credential"],
    "transaction_data_hashes_alg": "sha-256",
    "payload": {
        "transaction_id": "b0f75d4d-996b-46df-abb6-e3ddec390d2b",
        "display": {...}
    }
}
```

As a baseline, this specification defines schemas for the `payload` object for the following transaction types:

+ payment confirmation,
+ login and confirmation of actions carrying a risk of fraud or abuse, and
+ e-mandates.

## 3 Presentation Request Processing

For the Wallet Unit to determine whether a given attestation is an SCA Attestation, it SHALL evaluate the SD-JWT VC Type Metadata as per [SD-JWT-VC] section 6. If the VC Type Metadata contains a top-level claim `category` with the value `urn:eudi:sca`, it SHALL process the attestation as SCA Attestation according to this specification.

The following is a non-normative example of the VC Type Metadata including the `category` claim:

```json
{
    "vct": "https://pay.example.com/card",
    "name": "pay.example Payment Credential",
    "description": "This is a pay.example Payment Credential",
    "extends": "https://pay.example.com/basic-schema-1.2",
    "extends#integrity": "sha256-9cLlJNXN-TsMk-..5ca_xGgX3c1VLmXfh-WRL5",
    "schema_uri": "https://pay.example.com/card-schema-1.0",
    "schema_uri#integrity": "sha256-o984vn819a48..5t0WRL5ca_xGgX3c1VLmXfh",
    "category": "urn:eudi:sca"
}
```

### 3.1 Embedded Disclosure Policy and Signed Request Verification

SCA Attestation providers are RECOMMENDED to make use of embedded disclosure policies, as specified in the [ARF]. Details are to be specified in the respective SCA Attestation Rulebook.

Relying Parties sending a presentation request for an SCA Attestation SHOULD use signed Authorization Requests as specified in [OID4VP] and [HAIP] v1.0, section 5.

Upon receiving a presentation request, the Wallet Unit SHALL perform the following verifications:

1. **Disclosure Policy Verification**: If the SCA Attestation includes an embedded disclosure policy, the Wallet Unit **SHALL** evaluate it to verify that the presentation request was sent by a permitted Relying Party. If this verification fails, the Wallet Unit **SHALL** cease processing the request and inform the User.

2. **Signed Request Verification**: The Wallet Unit **SHALL** check if the presentation request is signed.
    - If it is signed, the Wallet Unit **SHALL** verify the signature and certificate. If this verification fails, the Wallet Unit **SHALL** cease processing the request and inform the User.
    - If the request is not signed, the Wallet Unit **SHALL** warn the User and require explicit confirmation before continuing to process the request.

### 3.2 Transactional Data Discovery and Validation

To ensure compatibility, Relying Parties **SHALL** only use transaction types and schemas that are specified in the `transaction_data_types` parameter of the SCA Attestation's type metadata.

Upon receiving a presentation request containing a `transaction_data` object, the Wallet Unit **SHALL** perform the following validation steps:

1. Verify that the `transaction_data.type` (e.g., `urn:eudi:sca:payment_authentication:1`) is a key in the `transaction_data_types` map found in the SCA Attestation's type metadata.
2. Retrieve the corresponding schema URI from the `schema` property for that transaction type.
3. Validate that the `transaction_data.payload` object conforms to the JSON schema referenced by the retrieved URI, or to one of the built-in transaction data types described in Section 4.2.
4. If any of these verifications fail, the Wallet Unit **SHALL** cease processing the request and inform the User with an appropriate error message.
5. If all verifications pass, the Wallet Unit **MAY** continue processing the presentation request.

The SCA Attestation metadata SHALL contain all the mandatory data elements specified in Section 4.1 and MAY contain additional elements specified in a related SCA Attestation Rulebook.

### 3.3 Transactional Data Processing

The transactional data are processed for the following purposes:

+ display message(s) rendering,
+ user consent proof generation and delivery,
+ other purposes not related to the two above, as specified in this document.


#### 3.3.1 Transaction Data Object

The Wallet Unit is provided with a `transaction_data` object in the [OID4VP] presentation request.

The `transaction_data` object related to an SCA transaction, SHALL contain all the mandatory data elements as specified in Section 4.2 and MAY contain additional data elements as specified in a related SCA Attestation Rulebook.

The specification defines the following mandatory parameters for the `payload` object:

+ **`transaction_id`**: An identifier for the payment transaction that requires consent from the Wallet Unit User.
+ **`display`**: An object containing parameters that the Wallet Unit **MUST** present to the user when obtaining consent for the SCA Attestation presentation. The Wallet Unit retrieves localized strings from the localization catalogue, which is specified in the `i18n` parameter of the SCA Attestation metadata.

The following is a non-normative example of the `transaction_data` object:

```json
{
    "type": "https://pay.example/trx/single_payment",
    "credential_ids": ["payment_credential"],
    "transaction_data_hashes_alg": "sha-256",
    "payload": {
        "transaction_id": "b0f75d4d-996b-46df-abb6-e3ddec390d2b",
        "display": {
            "payee": "Merchant XYZ",
            "amount": {
                "value": "100.00",
                "currency": "EUR"
            }
        }
    }
}
```

#### 3.3.2 Display Message Rendering

When requesting User consent for an SCA Attestation presentation, the Wallet Unit **SHALL** render a display message composed of two main groups of data elements:

1. **Attributes to be Presented**: This includes all attributes requested from the SCA Attestation and any other attestations. The rendering for this group **SHALL** follow the general presentation rules specified in the [ARF] for non-transactional data.
2. **Transactional Data**: This includes all parameters from the `display` object within the `transaction_data.payload`. The rendering for this group **SHALL** follow the rules specified in this document.

The Wallet Unit **SHALL** ensure that the User can review all data elements from both groups before providing consent. The payment transaction data (group 2) **SHALL** be visible on the screen when the User gives their consent.

The complete display message **SHALL** contain all mandatory data elements specified in Section 4.3 and **MAY** contain additional elements as specified in an SCA Attestation Rulebook.

The display message **SHALL** also comply with the following basic requirements for hierarchy and order:
_(Editor's notes: LSP representatives to provide a list of requirements here.)_

The Wallet Unit **MAY** comply with any additional display rendering requirements contained in an SCA Attestation Rulebook.


#### 3.3.3 Localisation

The localisation catalogue is a JSON object that provides localized strings for the user interface. It consists of key-value pairs with the following structure:

- **Key**: A string identifier that matches the `title` or `description` from the transaction data's JSON schema.
- **Value**: An array of objects, where each object **MUST** contain:
    - `lang`: A language identifier as defined in [BCP47].
    - `value`: The localized string for the specified language.

Each array of localized strings **MUST** include at least one entry where the `lang` parameter is set to `default`. The Wallet Unit **SHALL** use this entry as a fallback if a translation for the User's specific language is not available.

The localisation catalogue **SHALL** contain all mandatory data elements specified in Section 4.4 and **MAY** contain additional elements as specified in an SCA Attestation Rulebook.

The following is a non-normative example of the localisation catalogue object:

```json
{
    "single_payment.display.amount.title": [
        { 
            "lang": "de",
            "value": "Betrag"
        },
        { 
            "lang": "default",
            "value": "Amount"
        },
    ],
    "single_payment.display.currency.title": [
        { 
            "lang": "de",
            "value": "Währung"
        },
        { 
            "lang": "default",
            "value": "Currency"
        },
    ]
}
```


The Wallet Unit SHALL support a fallback mechnism, providing translations of the mandatory display elements (indicated in Section 4.4) in case the localisation catalogue object is not available.
_(Editor's note: to check if we need to specify more here)_

##### 3.3.3.1 UI Elements labels

The UI elements localization catalogue provides localized strings for UI components, such as buttons, used during a transaction. This catalogue is referenced by the `ui` parameter in the SCA Attestation type metadata.

The catalogue **SHALL** contain at least the following parameters but **MAY** be extended with additional parameters:

- **`affirmative_action_label`**: The label for the UI element used to confirm or consent to the action (e.g., "Confirm Payment").
- **`denial_action_label`**: The label for the UI element used to deny or cancel the action (e.g., "Cancel Payment").

The following is a non-normative example of an UI Elements localisation catalogue object:

```json
{
    "affirmative_action_label": [
        {
            "lang": "default",
            "value": "Confirm Payment"
        }
    ],
    "denial_action_label": [
        {
            "lang": "default",
            "value": "Cancel Payment"
        }
    ]
}
```

#### 3.3.4 User Consent Proof

Refer to section 3.5.


#### 3.4 Combined Presentation

Relying Party may request attributes from multiple attestations in a single [OID4VP] presentation request.
Therefore a presentation request can combine attributes from an SCA Attestation and other (non-SCA) attestations, enabling combination of payments and additional attributes presentation. Examples for such use cases could be payment plus age verification, driver's licence or loyalty cards.
The Digital Credentials Query Language (DCQL) allows the Relying Party to specify a combination of various (non-SCA Attestation related) attributes to facilitate the payment transaction. The details and examples on making DCQL queries in combined presentation request is presented in an SCA Attestation Rulebook.

_(Editor's note: to decide how other attributes are presented - if there are any differences than presenting SCA Attestation attributes.)_
_(Editor's note FA: this section is more relevant for the rendering rules in term of how to display it)_


### 3.5 Presentation Response

After the User gives consent, the Wallet Unit **SHALL** provide a response according to [OID4VP] and [HAIP]. This includes handling the `nonce` and `aud` claims as described in sections B.1.3.1.5 and B.3.6 of [OID4VP] v1.0.

The presentation of an SCA Attestation in the [SD-JWT-VC] format **SHALL** include a Key Binding JWT (KB-JWT) to ensure cryptographic binding between the User's action and the payment transaction, and to guarantee the freshness and uniqueness of the presentation.

In addition to the standard claims, this specification defines the following **REQUIRED** claims for the KB-JWT:

- **`jti`**: A fresh, cryptographically random value with sufficient entropy, as defined in [RFC7519], section 4.1.7. This value **MUST** be unique for each presentation. Once verified, it serves as the Authentication Code required by [PSD2] for electronic payments.
- **`authentication_factors`**: A JSON array that documents the authentication factors successfully applied by the Wallet Unit to authorize the presentation. This is required to satisfy the traceability requirements of the [PSD2] Regulatory Technical Standards (RTS).

The `authentication_factors` array **MUST** contain objects representing at least two different authentication categories. Each object consists of a key representing the category and a value specifying the method used:

- **Knowledge**: `{"knowledge": "PIN" | "passphrase" | "other"}`
- **Possession**: `{"possession": "WSCDSecuredKey" | "other"}`
- **Inherence**: `{"inherence": "fingerprint" | "face-device" | "face-external" | "other"}`

The `iat` (issued at) claim of the KB-JWT **MAY** be used by a Relying Party to restrict the timeframe in which the SCA result is accepted.

The following is a non-normative example of the KB-JWT object:

```json
{
    "aud": "x509_san_dns:shop.example.com",
    "iat": 1741269093,
    "jti": "deeec2b0-3bea-4477-bd5d-e3462a709481",
    "nonce": "bUtJdjJESWdmTWNjb011YQ",
    "sd_hash": "Re-CtLZfjGLErKy3eSriZ4bBx3AtUH5Q5wsWiiWKIwY",
    "authentication_factors": [
        {"knowledge": "PIN"},
        {"inherence": "fingerprint"}
    ],
    "transaction_data_hashes": [
        "OJcnQQByvV1iTYxiQQQx4dact-TNnSG-Ku_cs_6g55Q"
    ],
    "transaction_data_hashes_alg": "sha-256"
}
```

### 4 Data Elements and Structures

#### 4.1 SCA Attestation VC type Metadata Schema (tbd)

This section defines the structure and minimum set of parameters of SCA Attestation's metadata.


| Attribute | Optionality | Description |
|---------------|--------------|--------|
| `vct` | REQUIRED | As specified in [SD-JWT-VC] |
| `name` | REQUIRED |  As specified in [SD-JWT-VC] |
| `description` | REQUIRED | As specified in [SD-JWT-VC] |
| `schema_uri` | REQUIRED |  A URL pointing to a JSON Schema document describing the structure of the SCA Attestation. |
| `transaction_data_types` | REQUIRED |  An object containing a list of key/value pairs, where each key is a URI that identifies the type of a `transaction_data` object as defined by OpenID4VP that MAY be used during the presentation of the given SCA Attestation. The value is an object containing the following parameters:<li>`schema`: REQUIRED. URI pointing to a JSON Schema describing the structure of the `payload` object contained in the `transaction_data` object,</li><li>`i18n`: REQUIRED. URI pointing to a JSON document containing a catalogue of string identifiers and their localized representations for the descriptions of each field described by the schema.</li>  |

+ **`schema`**: A URI that references a JSON Schema defining the structure of the `payload` object within the `transaction_data` object of a given type.
+ **`i18n`**: A URI that references a JSON document containing a catalogue of string identifiers and their localized values for the descriptions of each field in the schema.
+ **`visualisation`**: A URI that references a JSON document containing a catalogue of string identifiers and their hierarchy level for visual representation on the confirmation screen.
+ **`ui`**: A URI that references a JSON document containing a catalogue of string identifiers and their localized values for UI elements, such as buttons, used in the context of the transaction.

#### 4.2 Transactional Data Object
The  transactional data is a JSON object contained in the `transaction_data` parameter of an [OID4VP] request. This section defines the structure and minimum set of parameters of transactional data object.


| Attribute | Optionality |      Description |
|---------------|--------------|--------|
| `type` | REQUIRED |  As specified in [OID4VP] |
| `credential_ids` | REQUIRED |  As specified in [OID4VP] |
| `transaction_data_hashes_alg` | REQUIRED |  As specified in [OID4VP] |
| `payload` | REQUIRED |  Object containing the details for the transaction. The object is described by the JSON schema referenced in the `schema` parameter given by the attestation's type metatada |

Wallet units **MUST** support the processing and rendering of the three basic transaction types defined in this specification. Support for additional transaction types is **OPTIONAL**. For each of the basic types, this specification provides the corresponding JSON schema files and fallback localization catalogues.

- `urn:eudi:sca:payment_authentication:1`
- `urn:eudi:sca:login_risk_transaction:1`
- `urn:eudi:sca:emandate:1`

#### 4.3 Payload Object

The `payload` object contains all the details for a specific transaction type.

#### 4.3.1 Payment Authentication

The `payload` object for the transaction data type `urn:eudi:sca:payment_authentication:1`.

- **`transaction_id`**: REQUIRED (string) Unique identifier of the transaction.
- **`payee_id`**: REQUIRED (string) An identifier of the payee that is understood by the payment system used to process the transaction.
- **`display`**: REQUIRED (object)
  - **`execution_date`**: REQUIRED (string, date_time) Execution date of the (first) payment. Absence of this optional attribute indicates immediate execution.
  - **`payee`**: REQUIRED (string) The name of the payee to whom the payment is being made.
  - **`amount`**: REQUIRED (object)
    - **`value`**: REQUIRED (number) Amount of the single payment, each recurring payment or the initial recurring payment in case of variable amounts.
    - **`currency`**: REQUIRED (string) Currency of the payment(s).
  - **`recurring`**: OPTIONAL (object) If present, it indicates a recurring payment.
    - **`occurrences`**: CONDITIONAL (integer) The number of occurrences of individual payment. Required when “apr” is present, else optional. Absence of this attribute (when optional) indicates unlimited occurrences.
    - **`total_amount`**: CONDITIONAL (number) The total amount of all payments under this transaction. Required when “occurrences” is present, else optional.
    - **`min_distance`**: REQUIRED (integer) The minimum number of days between single occurrences of the recurring schedule.
    - **`variable_amount`**: OPTIONAL (boolean) Indicates if subsequent transactions may have a different amount compared to the first transaction. Absence of this optional attribute indicates that the amount does not vary.
    - **`apr`**: REQUIRED (number) Annual Percentage Rate of the instalment. Presence of this optional attribute indicates that the transaction is an interest-bearing instalment.


#### 4.3.3 Login and Risk-based Authentication

The `payload` object for the transaction data type `urn:eudi:sca:login_risk_transaction:1`.

- **`transaction_id`**: OPTIONAL (string) Unique identifier of the transaction.
- **`display`**: REQUIRED (object)
  - **`date_time`**: REQUIRED (string, date_time) Date and time of the request for SCA.
  - **`service`**: OPTIONAL (string) The name of the service which triggered the operation (*e.g. Superbank Onlinebanking*).
  - **`action`**: REQUIRED (string) Description of the action (*e.g Login to Online-Banking or Change daily account limit from 1.000 EUR to 10.000 EUR*).

#### 4.3.4 eMandate

The `payload` object for the transaction data type `urn:eudi:sca:emandate:1`.

- **`transaction_id`**: OPTIONAL (string)
- **`display`**: REQUIRED (object)
  - **`date_time`**: REQUIRED (string, date_time) Date and time of the request for the mandate.
  - **`reference`**: OPTIONAL (string) A unique ID assigned to the specific mandate for tracking.
  - **`purpose`**: REQUIRED (string)
  - **`nbf`**: OPTIONAL (string, date-time) Date when the Mandate becomes valid.
  - **`exp`**: OPTIONAL (string, date-time) Expiration date of the mandate.
  - **`execution_date`**: OPTIONAL (string, date_time) Execution date of the (first) payment. Absence of this optional attribute indicates immediate execution.
  - **`amount`**: OPTIONAL (object)
    - **`value`**: REQUIRED (number) Amount of the single payment, each recurring payment or the initial recurring payment in case of variable amounts.
    - **`currency`**: REQUIRED (string) Currency of the payment(s).
  - **`recurring`**: OPTIONAL (object) May be used to indicate a recurring pull payment.
    - **`occurrences`**: OPTIONAL (integer) The number of occurrences of individual payment. Absence of this attribute (when optional) indicates unlimited occurrences.
    - **`total_amount`**: CONDITIONAL (number) The total amount of all payments under this transaction. Required when “occurrences” is present, else optional.
    - **`min_distance`**: REQUIRED (integer) The minimum number of days between single occurrences of the recurring schedule.
    - **`variable_amount`**: OPTIONAL (boolean) Indicates if subsequent transactions may have a different amount compared to the first transaction. Absence of this optional attribute indicates that the amount does not vary.



### 5 Other Requirements

#### 5.1 SCA Attestation Issuance

Note: This section is informative. The policy and binding requirements in this respect may be included in SCA Attestation Rulebooks.

The SCA Attestation Provider SHALL validate WUA at the issuance of an SCA Attestation. The SCA Attestation Provider SHALL issue the SCA Attestation if and only if the WUA is valid. _(Editor's note: this is already a generic requirement in ARF applicable for any attestation.)_

The SCA Attestation Provider SHALL request from the Wallet Unit to present a WUA that contains revocation information and with validity of at least one month.
_(Editor's note: Such a mechanism is not yet specified, this is to be aligned with TS3 possibly.)_

The SCA Attestation Providers SHALL issue the SCA Attestation with the validity time not longer than WUA.
_(Editor's note: this is already a generic requirement in ARF applicable for any attestation.)_


#### 5.2 SCA Attestation Revocation
Note: This section is informative. The policy and binding requirements in this respect may be included in a related SCA Attestation Rulebook.

The SCA Attestation Provider SHALL check regularly the Wallet Units validity (validity of WUAs) and invalidate the issued SCA Attestations for the revoked Wallet Units.

The Relying Party (requestor), before requesting presentation of a SCA Attestation (and thus initiating payment authentication process), SHALL verify validity of the SCA Attestation.
If the result of this verification is negative, the Relying Party SHALL cease the process.

### 5.3 SCA Attestation Validity Verification  

The Relying Party before sending a request for presentation of a SCA Attestation SHALL verify the validity of that SCA Attestation with the corresponding Trusted List (in case of a qualified attestation) or a URL provided in the associated SCA Attestation Rulebook.  

In case of negative result or unsuccessful verification, the Relying Party SHOULD cease the processing of the request.


### 6 References

| Reference      | Description |
|----------------|-------------|
| [Regulation (EU) No 910/2014] | [Regulation (EU) No 910/2014 of the European Parliament and of the Council of 23 July 2014 on electronic identification and trust services for electronic transactions in the internal market and repealing Directive 1999/93/EC](https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:32014R0910) |
| [Regulation (EU) 2024/1183] | [Regulation (EU) No 910/2014 of the European Parliament and of the Council of of 11 April 2024 amending Regulation (EU) No 910/2014 as regards establishing the European Digital Identity Framework](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex%3A32024R1183)  |
| PSD2 |[Directive (EU) 2015/2366 of the European Parliament and of the Council of 25 November 2015 on payment services in the internal market [...]](https://eur-lex.europa.eu/eli/dir/2015/2366/oj/eng) |
| [ARF]      | [Architecture and Reference Framework](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/architecture-and-reference-framework-main.md) |
| [JWE]      | [JSON Web Encryption (JWE), RFC 7516](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/433) |
| [JWA]      | [JSON Web Algorithms (JWA), RFC 7518](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/446) |
| [JSON]      | [The JavaScript Object Notation (JSON) Data Interchange Format, RFC 8259](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/79) |
| [SD-JWT-VC]      | [SD-JWT-based Verifiable Credentials (SD-JWT VC)](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/9)|
| [ISO/IEC 18013-5]  | [ISO/IEC 18013-5 - Mobile driving licence (mDL) application, First edition, 2021-09](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/84) |
| [OID4VP] |    [OpenID for Verifiable Presentations 1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/2)    |
| [OID4VCI] |    [OpenID for Verifiable Credential Issuance 1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3)    |
| [HAIP]  | [OIDF OpenID4VC High Assurance Interoperability Profile 1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/4)    |


