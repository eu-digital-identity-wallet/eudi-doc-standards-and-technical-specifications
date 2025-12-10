
<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification of Strong Customer Authentication (SCA) Implementation with the Wallet

## Abstract

The present document specifies data structures and processing rules for conducting Strong Customer Authentication (SCA) according to [PSD2] with use of the European Digital Identity Wallets.

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
| `0.96`   | 14.11.2025  | Further fine-tuning and error corrections |
| `0.97`   | 17.11.2025  | Adding transaction logging requirements |
| `0.98`   | 27.11.2025  | API files names update |
| `1.0`   | 5.12.2025  | Final version  |

## 1 Introduction

### 1.1 Overview

Strong Customer Authentication (SCA) for payments is mandated under [PSD2]  Article 97 (1) for electronic payment initiation, as well as for accessing a payment account online or other User's actions that present a risk of fraud. By design, the EUDI Wallet supports all three authentication factor categories required by [PSD2] (knowledge, possession, inherence) to enable SCA, as well as Dynamic Linking.

To ensure interoperability, a set of requirements is specified for the implementation of SCA using the Wallet. This technical specification is based on the discussion within Topics W and AA.

### 1.2 Scope of the Document

The [ARF] Section 2.6.4 introduces the concept of dedicated SUA Attestations issued into the User's Wallet Unit by their ASPSP and then later presented in conjunction with transactional data to be signed by the Wallet Unit (for Dynamic Linking).

In general, this specification aims to define the behaviour of the Wallet Units when receiving a presentation request of attributes from a SUA Attestation used for SCA (hereinafter called "SCA Attestation"). Therefore, the Wallet Solution Providers shall implement their Wallet Solutions in compliance with this document.

At the same time, the related SCA Attestation Rulebooks, delivered by the financial sector, provide all the necessary information and requirements for:

- Attestation Providers (Payment Service Providers, such as banks) to issue their SCA Attestations,
- Relying Parties (Payment Service Providers or other payment actors, such as Payees or Payment Initiation Service Providers) to make  presentation requests of the SCA Attestation, as payment transaction authentication.
Therefore the Attestation Providers and Relying Parties shall also follow a related SCA Attestation Rulebook.

This document contains:

- overview of the SCA scenarios at stake,
- minimum requirements and specifications for transactional data sent in a presentation request,
- minimum requirements and specifications for metadata of the SCA Attestation,
- transactional data processing and display message rendering requirements,
- other necessary requirements related to Wallet Solutions' functionality, interoperability or security.

This document is complemented by related SCA Attestation Rulebooks, that may specify:

- SCA Attestation types, applicable in various contexts,
- full specification of attributes of SCA Attestations,
- extended specifications of metadata of SCA Attestations,
- extended specifications of transactional data transported by the presentation protocol,
- extended display messages rendering rules,
- other requirements such as policies for SCA Attestation Providers or Relying Parties.

This document provides the minimum necessary requirements and specifications for the basic SCA flow, to be supported by all the Wallet Solutions. Therefore, the SCA Attestation Rulebooks shall be created in conformity with this specification.
The SCA Attestation Rulebooks may provide additional requirements and specifications extending this document (for instance to improve user experience or customise the scenarios) that may be implemented by Wallet Solutions on a voluntary basis.  

This version of the document focuses on [SD-JWT-VC] format and [OID4VP] presentation protocol only.

### 1.3 Requirements Notation

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",  "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in RFC2119 and RFC8174 when, and only when, they are written in all capital letters.

### 1.4 Terminology

| Term (Acronym)     | Definition   | Based on  |
|--------------------|--------------|-----------|
| Strong Customer Authentication (SCA) | An authentication based on the use of at least two authentication factors from different categories of either *knowledge* (something only the User knows), *possession* (something only the User possesses) or *inherence* (something the User is), that are independent, in that the breach of one does not compromise the reliability of the others, and is designed in such a way as to protect the confidentiality of the authentication data. | [PSD2] Article 97, [PSD2-RTS] Articles 4 ff. |
| Dynamic Linking | Strong Customer Authentication that includes elements which dynamically link the transaction to a specific amount and a specific Payee. | [PSD2] Article 97, [PSD2-RTS] Article 5 |
| Payment Service Provider (PSP) | A credit institution, electronic money institution, post office giro institution, or payment institution offering payment services to Payers and/or Payees. | [PSD2] Article 4 (11) |
| Account Servicing Payment Service Provider (ASPSP); synonyms: Issuer, Payer's PSP | Payment Service Provider providing and maintaining a payment account for a Payer and issuing the SCA Attestation into his/her Wallet Unit. | [PSD2] Article 4 (17) |
| Account Information Service Provider (AISP) | Payment Service Provider providing an online service to provide consolidated information on one or more payment accounts held by the Payment Service User with either another Payment Service Provider or with more than one Payment Service Provider. | [PSD2] Article 4 (16) and (19) |
| Payment Initiation Service Provider (PISP) | Payment Service Provider providing a service to initiate a payment order at the request of the Payment Service User with respect to a payment account held at another Payment Service Provider. | [PSD2] Article 4 (15) and (18) |
| Third-party Provider (TPP) | An AISP or a PISP. | |
| Open Banking | The entirety of services delivered by TPPs to PSUs and Payees using infrastructure that connects ASPSPs and TPPs. | |
| European Digital Identity Wallet (EUDIW); synonyms: EUDI Wallet, Wallet | An electronic identification means which allows the User to securely store, manage and validate person identification data and electronic attestations of attributes for the purpose of providing them to Relying Parties and other Users of European Digital Identity Wallets. | [eIDAS] Article 3 (42) |
| Payee | A natural or legal person who is the intended recipient of funds which have been the subject of a payment transaction. | [PSD2] Article 4 (9) |
| Payer; synonyms: User, Holder, Account Holder, Payment Service User (PSU) | A natural person who holds a payment account at a Payment Service Provider and who uses an EUDIW provided in accordance with [eIDAS]. | [PSD2] Article 4 (8), [eIDAS] Article 3 (5a) |
| SCA Attestation | An SUA Attestation used for SCA. |  |
| Merchant | A type of Payee, typically offering goods or services to Users. |  |

Further capitalised words and terms, not included in the above table, are to be interpreted in line with [ARF] Annex 1.

## 2 High-level Solution Design

_**Note**: This chapter is **informative**. It aims to help the reader to understand the context for this specification. All the possible scenarios and flows supported by this specification may be described in a related SCA Attestation Rulebook._  

### 2.1 Process Flows

On a high level, the solution is composed of the following parts:

- **Registration**: The secure association of the Wallet Unit and Holder with the Account Holder by means of the Payer's PSP issuing a dedicated, cryptographically bound SCA Attestation into the User's Wallet Unit. This step is not described in this document.
- **Authentication**: The Strong Customer Authentication at time of transaction by means of presenting the SCA Attestation.
- **Dynamic Linking**: Linking of the User's consent to a specific transaction by means of including transaction-specific information in the presentation request and (signed) response.

Using an EUDI Wallet for SCA in general can happen in one of two operating setups:

- In the first setup, the Wallet Unit is presenting the SCA Attestation back to the Attestation Provider, usually the User's PSP (or an Intermediary on their behalf). The User may be entirely in the context of their PSP (e.g., interacting with their online/mobile banking to send a credit transfer) or he/she may have been redirected from a third party (e.g., a Merchant or a TPP) to the PSP's environment. When used like this, the EUDIW resembles banks' existing authenticator solutions. We refer to this setup as the "**Issuer-requested flow**".
- In the second setup, the Wallet Unit is presenting the SCA Attestation to a third party which is not the User's PSP (or an Intermediary on their behalf). We refer to this setup as the "**Third-party-requested flow**". It is driven by two constellations:
  - As per [PSD2], the User may give consent to execute a payment not only directly towards their PSP, but also via the Payee (or an Intermediary on their behalf). When used like this, the EUDIW loosely resembles an xPay wallet functionality.
  - The integration models for TPPs such as PISPs and AISPs know a variant called "embedded SCA flow" where the User provides authentication information via the TPP to the ASPSP.

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

An SCA Attestation can be requested for various transaction types, such as logging into online banking or confirming a payment. To prevent fraudulent abuse, Attestation Providers must be able to control which combinations of transaction types and SCA Attestation types are permitted (e.g., for Issuer-requested vs. Third-party-requested flow variants). To enable this control, this specification extends the standard SD-JWT Verifiable Credential (VC) type metadata functionality of [SD-JWT-VC].

This specification introduces a `transaction_data_types` parameter to the [SD-JWT-VC] type metadata. This parameter contains an entry for each permitted transaction data type associated with a given SCA Attestation type. Each entry, in turn, contains information that defines how to process different aspects of the transaction type when presenting an SCA Attestation.

These aspects comprise the following:

- **Transaction data schema**: [JSON Schema] describing the data structure of the transactional data's content.
- **Localisation of the data schema**: JSON object containing localisation strings for the data fields of the transactional data.
- **Visualisation**: A visualisation hierarchy level indicating the importance of individual elements of the transactional data.
- **Wallet UI Elements**: JSON object containing localisation strings for different UI elements used by the wallet during a transaction.

The JSON schema and data objects can be embedded or referenced by URI. In case of URNs (non-resolvable), the Wallet Solution recognises them through this or another specification it adheres to. In case of URLs (resolvable), the Wallet Unit will resolve the address to obtain and process the data from that location.

Attestation Providers are expected to use collision-resistant identifiers for transaction data types. The method for communicating these identifiers to the party assembling the transactional data (e.g., the RP) is beyond the scope of this document. It is expected that sector-specific standardisation bodies or schemes will provide this governance and/or offer automated services for RPs.

The following is a non-normative example of an SCA Attestation type metadata:

```json
{
    "vct": "https://pay.example.com/card",
    "name": "pay.example Payment Credential",
    "description": "This is a pay.example Payment Credential",
    "extends": "https://pay.example.com/basic-schema-1.2",
    "schema_uri": "https://pay.example.com/card-schema-1.0",
    "transaction_data_types": {
        "https://standardsbody.org/eudiw-trx/payment": {
            "schema": "urn:eudi:sca:payment:1",
            "claims": [
                {
                    "path": [ "payload", "transaction_id"],
                    "visualisation": 4,
                    "display": [
                        {
                            "lang": "de-DE",
                            "label": "Transaktionsnummer",
                            "description": "Eindeutige Nummer der Transaktion"
                        },
                        {
                            "lang": "en-GB",
                            "label": "Transaction ID",
                            "description": "Unique identifier of the transaction"
                        }
                    ]
                }
                ...
            ],
            "ui_labels_uri" : "https://pay.example.com/ui-localisation-catalogue.json"
        }
    }
}
```

The detailed specification for these parameters is located in Section 4. An SCA Attestation Rulebook may specify additional parameters for optional support by Wallet Solutions.

The SCA Attestation Provider shall publish the metadata parameters in accordance with the specification provided in [SD-JWT-VC].

### 2.4 Standardised Transaction Data

To foster interoperability between wallet providers and banks, this specification defines the data schemas for basic transaction types every Wallet Unit needs to support in terms of rendering and processing in order to accommodate the following use cases:

- payment confirmation,
- login and confirmation of actions carrying a risk of fraud or abuse,
- accessing payment account information, and
- e-mandates for Payee-initiated transactions.

To do so, this specification introduces and standardises a `payload` object under [OID4VP]'s `transaction_data`.

## 3 Presentation Request Processing

For the Wallet Unit to determine whether a given attestation is an SCA Attestation, it **SHALL** evaluate the SD-JWT VC Type Metadata as per [SD-JWT-VC] section 6. If the VC Type Metadata contains a top-level claim `category` with the value `urn:eu:europa:ec:eudi:sua:sca`, it **SHALL** process the attestation as SCA Attestation according to this specification.

The following is a non-normative example of the VC Type Metadata including the `category` claim:

```json
{
    "vct": "https://pay.example.com/card",
    "name": "pay.example Payment Credential",
    "description": "This is a pay.example Payment Credential",
    "extends": "https://pay.example.com/basic-schema-1.2",
    "schema_uri": "https://pay.example.com/card-schema-1.0",
    "category": "urn:eu:europa:ec:eudi:sua:sca"
}
```

### 3.1 Embedded Disclosure Policy and Signed Request Verification

SCA Attestation providers are **RECOMMENDED** to make use of embedded disclosure policies, as specified in the [ARF]. Details are to be specified in an SCA Attestation Rulebook.

Relying Parties sending a presentation request for an SCA Attestation **SHOULD** use signed "Authorization Requests" as specified in [OID4VP] and [HAIP] section 5.

Upon receiving a presentation request, the Wallet Unit **SHALL** perform the following verifications:

1. **Disclosure Policy Verification**: If the SCA Attestation includes an embedded disclosure policy, the Wallet Unit **SHALL** evaluate it to verify that the presentation request was sent by a permitted Relying Party. If this verification fails, the Wallet Unit **SHALL** cease processing the request and inform the User.

2. **Signed Request Verification**: The Wallet Unit **SHALL** check if the "Authorization Request" is signed.
    - If it is signed, the Wallet Unit **SHALL** verify the signature and certificate. If this verification fails, the Wallet Unit **SHALL** cease processing the request and inform the User.
    - If the request is not signed, the Wallet Unit **SHALL** warn the User and require explicit confirmation before continuing to process the request.

### 3.2 Transactional Data Discovery and Validation

To ensure compatibility, Relying Parties **SHALL** only use transaction types and schemas that are specified in the `transaction_data_types` parameter of the SCA Attestation's type metadata.

Upon receiving a presentation request containing a `transaction_data` object for a specific attestation type, the Wallet Unit **SHALL** perform the following validation steps:
1. Verify that the requested attestation type is an SCA Attestation of the `category` `urn:eu:europa:ec:eudi:sua:sca`.
2. Verify that the `transaction_data.type` (e.g., `https://standardsbody.org/eudiw-trx/payment`) is a key in the `transaction_data_types` map found in the SCA Attestation's type metadata.
3. Retrieve the corresponding schema URI from the `schema` property for that transaction type.
4. Validate that the `transaction_data.payload` object conforms to the [JSON Schema] referenced by the retrieved URI, or to one of the built-in transaction data types described in Section 4.2.
5. If any of these verifications fail, the Wallet Unit **SHALL** cease processing the request and inform the User with an appropriate error message.
6. If all verifications pass, the Wallet Unit **MAY** continue processing the presentation request.

The SCA Attestation metadata **SHALL** contain all the mandatory data elements specified in Section 4.1 and **MAY** contain additional elements specified in a related SCA Attestation Rulebook.

### 3.3 Transactional Data Processing

The transactional data are processed for the following purposes:

- display message(s) rendering,
- User consent proof generation and delivery,
- other purposes not related to the two above, as specified in this document.

#### 3.3.1 Display Message Rendering

When requesting User consent for an SCA Attestation presentation, the Wallet Unit **SHALL** render a display message composed of three main sources of data elements:

1. **Attributes to be Presented**: This includes all attributes requested from the SCA Attestation and any other attestations. The rendering for this data **SHALL** follow the general presentation rules specified in the [ARF] (as for presentations not involving transactional data). In case the Attestation Provider uses the `display` functionality for `credential_metadata` as defined in [OID4VCI] or the `display` metadata as defined in [SD-JWT-VC], the Wallet Unit **SHALL** adhere to these instructions.
2. **Transactional Data**: This includes all parameters from the `payload` object within the `transaction_data`. The rendering for this group **SHALL** follow the rules specified in this document.
3. **UI Labels**: This includes the localised labels for the UI elements introduced in this specification.

Wallet Solutions **SHALL** ensure that Users are able to give explicit and informed consent to a transaction. The Wallet Unit **SHALL** therefore display transactional data (or parts of it, see below) in conjunction with the requested attributes of the SCA Attestation to the User in a clear, understandable and accurate manner for review before obtaining the User's confirmation for the transaction. The Wallet Unit **SHALL** populate the UI Elements specified in Section 3.3.3 with the (localised) labels provided in the SCA Attestation type's metadata. In case the transactional data's localised parameter names or the required UI elements' localised labels are not available to the Wallet Unit at the time of transaction, the Wallet Unit **SHALL** cease processing the transaction and inform the User.

For transaction data, this specification defines hierarchy levels of visual importance that **SHALL** be implemented by Wallet Solutions. SCA Attestation Providers are **RECOMMENDED** to set these levels for leaf parameter names and their values within the `payload` object through the `visualisation` parameter of the Transaction Data Claim Metadata as introduced in Section 3.3.2. 

The levels are:

- `1`: The parameter name and value **MUST** be _prominently_ displayed on the main transaction confirmation screen.
- `2`: The parameter name and value **MUST** be displayed on the main transaction confirmation screen.
- `3`: The parameter name and value **MUST** be displayed, but **MAY** be displayed on a supplementary or subsequent transaction confirmation screen, including where User action is needed to invoke this screen.
- `4`: The parameter name and value **MAY** be omitted from being displayed on the transaction confirmation screen(s).

In case no `visualisation` level is set in the metadata for a given leaf parameter, the Wallet Unit **SHALL** apply a default value of `3`.

SCA Attestation Rulebooks **MAY** permit to visualise parameter names as icons or to have them omitted from being displayed if the User can easily understand the nature of the parameter from the parameter value itself. Wallet Solutions **SHALL** therefore support the generic functionality to show icons instead of parameter names or to omit them from being displayed.

The Wallet Unit **MAY** comply with any additional display rendering requirements contained in an SCA Attestation Rulebook.

#### 3.3.2 Transaction Data Claim Metadata

The SCA Attestation type metadata **MUST** also supply metadata for each claim of the transaction data object payload in the same manner as it is specified in the section "Claim Metadata" of [SD-JWT-VC] using the `claims` parameter. The claim metadata **MUST** be used to provide human readable labels and descriptions in multiple languages and also a hierarchy level for the visual representation on the confirmation screen.

This specification extends the Claim Metadata of [SD-JWT-VC] by the following parameter:  

- `visualisation`: **OPTIONAL** (Integer) A hierarchy level between 1 and 4 for visual representation on the confirmation screen.

The `sd` parameter is not applicable in the case of Transaction Data Claim Metadata.

#### 3.3.3 UI Elements Catalogue

The localisation catalogue for UI elements is a JSON object that provides localised strings for the following UI components used during a transaction. If provided, Wallet Solutions **SHALL** label these UI elements with the respective value.

- **`affirmative_action_label`**: **REQUIRED** The label for the UI element used to confirm or consent to the action (e.g., "Confirm Payment").
- **`denial_action_label`**: **OPTIONAL** The label for the UI element used to deny or cancel the action (e.g., "Cancel Payment"). In the absence of this element the Wallet Solution **SHALL** define a label for the UI element to deny or cancel the action.
- **`transaction_title`**: **OPTIONAL** A title/headline for the transaction confirmation screen which the Wallet Unit displays to the User. In the absence of this element the Wallet Solution **MAY** define a title.
- **`security_hint`**: **OPTIONAL** A security hint to be displayed to the User. In the absence of this element the Wallet Unit **SHALL NOT** display the security hint.

Additional UI elements identifiers **MAY** be defined if supported by a Wallet Solution.

The catalogue **SHALL** be provided by the Attestation Provider as a list of key-value pairs with the following structure:

- **Key**: UI element identifier
- **Value**: An array of objects, where each object **SHALL** contain:
  - `lang`: A language identifier as defined in [RFC5646].
  - `value`: The localised string for the specified language. The following maximum string lengths apply for the elements defined in this specification:
    - `affirmative_action_label`: 30 characters,
    - `denial_action_label`: 30 characters,
    - `transaction_title`: 50 characters,
    - `security_hint`: 250 characters.

This catalogue is either referenced by the `ui_labels_uri` parameter or included in the `ui_labels` parameter in the SCA Attestation Transaction type metadata.

The following is a non-normative example of an UI elements localisation catalogue object:

```json
{
    "affirmative_action_label": [
        {
            "lang": "de",
            "value": "Zahlung bestätigen"
        },
        {
            "lang": "en",
            "value": "Confirm Payment"
        }
    ],
    "denial_action_label": [
        {
            "lang": "de",
            "value": "Zahlung abbrechen"
        },
        {
            "lang": "en",
            "value": "Cancel Payment"
        }
    ]
}
```

### 3.4 Combined Presentation

Relying Parties may request attributes from multiple attestations in a single presentation request.
Therefore a presentation request can combine attributes from an SCA Attestation and other (non-SCA) attestations, enabling combination of payments and additional attributes presentation. Examples for such use cases could be payment plus age verification, driver's licence or loyalty cards.
The Digital Credentials Query Language (DCQL) allows the Relying Party to specify a combination of various (non-SCA Attestation related) attributes to facilitate the payment transaction. The details and examples on making DCQL queries in combined presentation requests **MAY** be presented in an SCA Attestation Rulebook.

### 3.5 Presentation Request Encryption

The SCA Attestation presentation request may carry (sensitive) personal data contained within the transactional data. To mitigate risks related to privacy, it is **RECOMMENDED** for Relying Parties to use encrypted presentation requests.
Therefore, the Wallet Units **SHALL** support processing of encrypted presentation requests in compliance with [JAR] and [OID4VP] Section 5.10.

### 3.6 Presentation Response

After the User gives consent, the Wallet Unit **SHALL** provide a response according to [OID4VP] and [HAIP]. This includes handling the `nonce` and `aud` claims as described in sections B.1.3.1.5 and B.3.6 of [OID4VP] v1.0.

The presentation of an SCA Attestation in the [SD-JWT-VC] format **SHALL** include a Key Binding JWT (KB-JWT) to ensure cryptographic binding between the User's action and the transaction, and to guarantee the freshness and uniqueness of the presentation.

In addition to the standard claims, this specification defines the following claims for the KB-JWT:

- **`jti`**: **REQUIRED** A fresh, cryptographically random value with sufficient entropy, as defined in [RFC7519], section 4.1.7. This value **SHALL** be unique for each presentation. Once verified, it serves as the Authentication Code required by [PSD2] for electronic payments.
- **`response_mode`**: **REQUIRED** The value of the `response_mode` parameter given in the [OID4VP] Authorization Request. The value may be used to evaluate the [OID4VP] implementation of a Relying Part in case the Presentation is used in a Third-party-requested flow.
- **`amr`**: **REQUIRED** Authentication Methods References. A JSON array that documents the authentication factors successfully applied by the Wallet Unit to authorize the presentation. This is required to satisfy the traceability requirements of the [PSD2] Regulatory Technical Standards (RTS).

The `amr` array **SHALL** contain at least two different objects representing authentication categories. Each object consists of a key representing the category and a value specifying the method used. The following keys and values are defined by this specification:

- **`knowledge`**:
  - `pin_less_than_6_digits`
  - `pin_6_or_more_digits`
  - `passphrase_less_than_8_chars`
  - `passphrase_8_to_11_chars`
  - `passphrase_12_or_more_chars`
  - `pattern`
  - `other`
- **`possession`**:
  - `key_in_remote_wscd`
  - `key_in_local_external_wscd`
  - `key_in_local_internal_wscd`
  - `key_in_local_native_wscd`
  - `other`
- **`inherence`**:
  - `fingerprint_device`
  - `fingerprint_external`
  - `face_device`
  - `face_external`
  - `other`

The `iat` (issued at) claim of the KB-JWT **MAY** be used by a Relying Party to restrict the timeframe in which the SCA result is accepted.

The following is a non-normative example of the KB-JWT object:

```json
{
    "aud": "x509_san_dns:shop.example.com",
    "iat": 1741269093,
    "jti": "deeec2b0-3bea-4477-bd5d-e3462a709481",
    "nonce": "bUtJdjJESWdmTWNjb011YQ",
    "sd_hash": "Re-CtLZfjGLErKy3eSriZ4bBx3AtUH5Q5wsWiiWKIwY",
    "amr": [
        {"knowledge": "passphrase_less_than_8_chars"},
        {"inherence": "fingerprint_device"}
    ],
    "transaction_data_hashes": [
        "OJcnQQByvV1iTYxiQQQx4dact-TNnSG-Ku_cs_6g55Q"
    ],
    "transaction_data_hashes_alg": "sha-256"
}
```

## 4 Data Elements and Structures

### 4.1 SCA Attestation VC Type Metadata Schema

This section defines the claims for SCA Attestation's metadata in addition to the claims specified for SD-JWT VC Type Metadata in [SD-JWT-VC].

- **`category`**: **OPTIONAL** (string) Category the attestation belongs to. **MAY** be used to indicate to the Wallet Unit that the attestation needs to be processed as the SCA Attestation.
- **`transaction_data_types`**: **REQUIRED** (string) An object containing a list of key/value pairs:
  - **`key`**: **REQUIRED** (string) URI that identifies the type of a `transaction_data` object as defined by OpenID4VP that **MAY** be used during the presentation of the given SCA Attestation.
  - **`value`**: **REQUIRED** (object)
    - **`schema`**: **CONDITIONAL** (string) Embedded [JSON Schema] defining the structure of the `payload` object within the `transaction_data` object of a given type. **MUST NOT** be used if `schema_uri` is present.
    - **`schema_uri`**: **CONDITIONAL** (string) URI that references a [JSON Schema] defining the structure of the `payload` object within the `transaction_data` object of a given type. **MUST NOT** be used if `schema` is present.
    - **`claims`**: **CONDITIONAL** Array of objects containing information about particular claims for displaying and validating the claims according to section 3.3.3. **MUST NOT** be used if `claims_uri` is present.
    - **`claims_uri`**: **CONDITIONAL** URI that references a JSON document containing an array of objects containing information about particular claims for displaying and validating the claims according to section 3.3.3. **MUST NOT** be used if `claims` is present.
    - **`ui_labels`**: **CONDITIONAL** Embedded JSON document containing a catalogue of string identifiers and their localised values for UI elements, such as buttons, used in the context of the transaction. **MUST NOT** be used if `ui_labels_uri` is present.
    - **`ui_labels_uri`**: **CONDITIONAL** URI that references a JSON document containing a catalogue of string identifiers and their localised values for UI elements, such as buttons, used in the context of the transaction. **MUST NOT** be used if `ui_labels` is present.

#### 4.1.1 Retrieving Metadata

Wallet Unit SHALL support metadata retrieval methods as specified in section 6.3.1. and 6.3.4. of [SD-JWT-VC].

#### 4.1.2 Metadata Integrity Verification

Section 7 of [SD-JWT-VC] introduces a mechanism for Wallet Units to verify the integrity of external metadata files referenced within an SCA Attestation. It is **RECOMMENDED** for SCA Attestation Providers to include this integrity mechanism in their SCA Attestations.

If this mechanism is used, the SCA Attestation Provider **SHALL** include a hash value of the referenced metadata file within the SCA Attestation claim. This claim **SHALL** be suffixed with the `#integrity` designator (e.g., `vct#integrity`). To address individual elements within the transaction data types array, the bracket notation`[]` **SHALL** be used.

If an SCA Attestation includes any `#integrity` suffixed claims, Wallet Unit **SHALL** verify the integrity of the retrieved document by comparing the calculated hash of the retrieved document against the value of the respective `#integrity` suffixed claim, as defined in section 7 of [SD-JWT-VC].

Wallet Unit **MAY** omit from displaying to User claims suffixed with `#integrity`.

The inclusion of the `#integrity` claims dictates that any modification to the SCA Attestation's type metadata will cause a negative result of the integrity check, thereby invalidating the existing SCA Attestation. In consequence, re-issuance of the SCA attestation is necessary.

Non-normative example for an SD-JWT VC payload including integrity claims for the transaction data type array:

```json
{
    "iss": "https://issuer.superbank3.com",
    "sub": "37774a3f-ab14-43c3-96bc-bb1066a30a1d",
    ...
    "vct": "https://psd2.standard.org/payment_service_user",
    "vct#integrity": "sha256-9cLlJNXN-TsMk-..5ca_xGgX3c1VLmXfh-WRL5",
    "transaction_data_types['https://standardsbody.org/eudiw-trx/payment'].schema_uri#integrity": "sha256-e3b0c44298fc49afbf4...a495991852b855",
    "transaction_data_types['https://standardsbody.org/eudiw-trx/payment'].ui_labels_uri#integrity": "sha256-335f3750519114d2a93...5e6ed24c30c0317"
    ...
}
```

### 4.2 Transactional Data Object

The transactional data is a JSON object contained in the `transaction_data` parameter of an [OID4VP] request. This section defines the structure and minimum set of parameters of transactional data objects.

This specification extends the `transaction_data` object defined in [OID4VP] by adding a `payload` parameter. This parameter is an object with the transaction details, and its structure is defined by the [JSON Schema] referenced in the `schema` parameter of the SCA Attestation type metadata.

- **`type`**: **REQUIRED** As specified in [OID4VP].
- **`credential_ids`**: **REQUIRED** As specified in [OID4VP].
- **`transaction_data_hashes_alg`**: **REQUIRED** As specified in [OID4VP].
- **`payload`**: **REQUIRED** A JSON object containing the details for the transaction. The object is described by the [JSON Schema] referenced in the `schema` parameter given by the attestation's type metadata.

### 4.3 Payload Object

The `payload` object contains all the details for a specific transaction type.

Wallet Units **SHALL** support the processing and rendering of the basic transaction types defined in this specification. Wallet Units **MAY** support additional transaction types, specified out of this specification. For each of the basic transaction types, this specification provides the corresponding [JSON Schema] files:

- [`urn:eudi:sca:payment:1`](./api/ts12-urn-eudi-sca-payment-1-data-model.json)
- [`urn:eudi:sca:login_risk_transaction:1`](./api/ts12-urn-eudi-sca-login_risk_transaction-1-data-model.json)
- [`urn:eudi:sca:account_access:1`](./api/ts12-urn-eudi-sca-account_access-1-data-model.json)
- [`urn:eudi:sca:emandate:1`](./api/ts12-urn-eudi-sca-emandate-1-data-model.json)

#### 4.3.1 Payment Confirmation

The content of the `payload` object for the transaction data type `urn:eudi:sca:payment:1`. This transaction data type is meant to be used in conjunction with an SCA Attestation that represents an account or a card.

- **`transaction_id`**: **REQUIRED** (string) Unique identifier of the Relying Party's interaction with the User.
- **`date_time`**: **OPTIONAL** (string) [ISO8601] date and time when the Relying Party started to interact with the User.
- **`payee`**: **REQUIRED** (object) Object holding the Payee details.
  - **`name`**: **REQUIRED** (string) Name of the Payee to whom the payment is being made.
  - **`id`**: **REQUIRED** (string) An identifier of the Payee that is understood by the payment system used to process the transaction.
  - **`logo`**: **OPTIONAL** (string) Resolvable or Data (as per [RFC2397]) URL of the Payee logo.
  - **`website`**: **OPTIONAL** (string) Resolvable URL of the Payee's website.
- **`pisp`**: **OPTIONAL** (object) If present, it indicates that the payment is being facilitated by a PISP.
  - **`legal_name`**: **REQUIRED** (string) Legal name of the PISP.
  - **`brand_name`**: **REQUIRED** (string) Brand name of the PISP.
  - **`domain_name`**: **REQUIRED** (string) Domain name of the PISP as secured by the [eIDAS] QWAC certificate of the TPP.
- **`execution_date`**: **CONDITIONAL**: **MUST NOT** be present when `recurrence` is present. **MAY** be present when `recurrence` is not present. (string) [ISO8601] date of the payment's execution. **MUST NOT** lie in the past. When permitted, absence of this optional attribute indicates immediate execution in case of a Payer-initiated payment (also called "customer-initiated transaction" or "CIT", or that the execution date is unknown at the time of SCA in case of a Payee-initiated payment (also called "merchant-initiated transactions" or "MIT").
- **`currency`**: **REQUIRED** (string) Currency of the payment(s) as [ISO4217] Alpha-3 code.
- **`amount`**: **REQUIRED** (number) Amount of the single payment, or regular amount of a recurring payment, consisting of major currency units as the integer component and an optional fraction part consisting of the decimal point followed by minor currency units, with the number of fractional digits as per [ISO4217]. Negative amounts are signed by minus.
- **`amount_estimated`**: **OPTIONAL** (boolean) In case of an MIT, indicates that the amount is estimated. Absence of this optional attribute indicates that the amount is not estimated.
- **`amount_earmarked`**: **OPTIONAL** (boolean) In case of an MIT that is not executed immediately, indicates that the Payee earmarks the amount immediately. Absence of this optional attribute indicates that no earmarking is taking place.
- **`sct_inst`**: **OPTIONAL** (boolean) Indicates that the ASPSP is requested to execute the payment as SEPA Instant Credit Transfer. Absence of this optional attribute indicates that no instant transfer has been requested.
- **`recurrence`**: **OPTIONAL** (object) If present, it indicates a recurring payment.
  - **`start_date`**: **OPTIONAL** (string) [ISO8601] date of the first payment's execution. This attribute is expected to be present in most cases, with the only exception of MITs when the date of the first payment due is unknown at the time of SCA.
  - **`end_date`**: **OPTIONAL** (string) [ISO8601] date of the last payment's execution.
  - **`number`**: **OPTIONAL** (integer) Number of recurring payments.
  - **`frequency`**: **REQUIRED** (string) The following codes are supported in line with [ISO20022]. Wallet Units **SHALL** provide human-readable, localised strings for these values:
    - `INDA` for intraday (i.e., several times a day),
    - `DAIL` for daily,
    - `WEEK` for weekly,
    - `TOWK` for every two weeks,
    - `TWMN` for twice a month,
    - `MNTH` for monthly,
    - `TOMN` for every two months,
    - `QUTR` for quarterly,
    - `FOMN` for every four months,
    - `SEMI` for twice a year,
    - `YEAR` for yearly,
    - `TYEA` for every two years.
  - **`mit_options`**: **OPTIONAL** (object) Object holding options for recurring MITs.
    - **`amount_variable`**: **OPTIONAL** (boolean) Indicates if subsequent transactions may have a different amount compared to the first transaction. Absence of this optional attribute indicates that the amount does not vary.
    - **`min_amount`**: **OPTIONAL** (number) The minimum amount of a single payment under this transaction. See `amount` for formatting.
    - **`max_amount`**: **OPTIONAL** (number) The maximum amount of a single payment under this transaction. See `amount` for formatting.
    - **`total_amount`**: **OPTIONAL** (number) The total amount of all payments under this transaction. See `amount` for formatting.
    - **`initial_amount`**: **OPTIONAL** (number) The deviating amount for a fixed number of initial instances of the recurring payment. See `amount` for formatting.
    - **`initial_amount_number`**: **OPTIONAL** (integer) The number of initial instances of the recurring payment with a deviating amount.
    - **`apr`**: **OPTIONAL** (number) Annual Percentage Rate of the installment. Presence of this attribute indicates that the transaction is an interest-bearing installment.

_**Note**: Payment networks may not support or support only some of the optional parameters of `payload` object. Therefore it is the requestor's (Relying Party's) responsibility to possess the knowledge about parameters that are not supported or required by a given payment network, to ensure that the payment transaction will be executed over the payment network in question. The Relying Party may consult these parameters with the SCA Attestation Providers or the payment networks._

#### 4.3.2 Login and Risk-based Authentication

The content of the `payload` object for the transaction data type `urn:eudi:sca:login_risk_transaction:1`. This transaction data type is meant to be used in conjunction with an SCA Attestation that represents a User.

- **`transaction_id`**: **REQUIRED** (string) Unique identifier of the Relying Party's interaction with the User.
- **`date_time`**: **OPTIONAL** (string) [ISO8601] date and time when the Relying Party started to interact with the User.
- **`service`**: **OPTIONAL** (string) The name of the service which triggered the operation (e.g. *"Superbank Online Banking"*).
- **`action`**: **REQUIRED** (string) Description of the action (e.g *"Log in to Online Banking"* or *"Change daily transaction limit from 1,000 EUR to 10,000 EUR"*).

#### 4.3.3 Payment Account Information Access

The content of the `payload` object for the transaction data type `urn:eudi:sca:account_access:1`. This transaction data type is meant to be used in conjunction with an SCA Attestation that represents an account.

- **`transaction_id`**: **REQUIRED** (string) Unique identifier of the Relying Party's interaction with the User.
- **`date_time`**: **OPTIONAL** (string) [ISO8601] date and time when the Relying Party started to interact with the User.
- **`aisp`**: **OPTIONAL** (object) If present, it indicates that the account (information) access is being facilitated by an AISP.
  - **`legal_name`**: **REQUIRED** (string) Legal name of the AISP.
  - **`brand_name`**: **REQUIRED** (string) Brand name of the AISP.
  - **`domain_name`**: **REQUIRED** (string) Domain name of the AISP as secured by the [eIDAS] QWAC certificate of the TPP.
- **`description`**: **OPTIONAL** (string) Description of the account (information) access the User is agreeing to.

#### 4.3.4 E-mandate

The content of the `payload` object for the transaction data type `urn:eudi:sca:emandate:1`. This transaction data type is meant to be used in conjunction with an SCA Attestation that represents an account or a card.

- **`transaction_id`**: **REQUIRED** (string) Unique identifier of the Relying Party's interaction with the User.
- **`date_time`**: **OPTIONAL** (string) [ISO8601] date and time when the Relying Party started to interact with the User.
- **`start_date`**: **OPTIONAL** (string) [ISO8601] date or date and time when the mandate becomes valid. Absence of this optional attribute indicates that the mandate becomes valid immediately.
- **`end_date`**: **OPTIONAL** (string) [ISO8601] date or date and time when the mandate expires. Absence of this optional attribute indicates that the mandate has no defined end date.
- **`reference_number`**: **OPTIONAL** (string) A unique ID assigned to the specific mandate. For a SEPA Direct Debit, this attribute takes the mandate reference number.
- **`creditor_id`**: **OPTIONAL** (string) For a SEPA Direct Debit, this attribute holds the creditor identifier.
- **`purpose`**: **CONDITIONAL**: **REQUIRED** when `payment` is not present, else **OPTIONAL** (string) Mandate text the User is supposed to consent to.
- **`payment_payload`**: **OPTIONAL** (object) Object of the type `urn:eudi:sca:payment:1` as defined in Section 4.3.1. The parameter **MAY** be used to leverage that data structure to obtain a mandate for MITs.

_**Note**: This transaction data type may get amended in a future version of this specification to also cover usages in the context of agentic commerce._

## 5 Other Requirements

### 5.1 SCA Attestation Issuance

_**Note**: This section is **informative**. The policy and binding requirements in this respect may be included in related SCA Attestation Rulebooks._

The SCA Attestation Provider shall validate the Wallet Unit Attestation (WUA) at the issuance of an SCA Attestation. The SCA Attestation Provider shall issue the SCA Attestation if and only if the WUA is valid.
(_**Note**: This is already a generic requirement in ARF applicable for any attestation._)

The SCA Attestation Provider shall request from the Wallet Unit to present a WUA that contains revocation information and with a validity period of at least one month.
(_**Note**: A mechanism for requesting by an Attestation Provider a WUA with a specific parameters is not yet specified; this is to be aligned with [TS3] possibly._)

The SCA Attestation Providers shall issue the SCA Attestation with a validity period no longer than WUA's one.
(_**Note**: this is already a generic requirement in ARF applicable for any attestation._)

### 5.2 SCA Attestation Revocation

_**Note**: This section is **informative**. The policy and binding requirements in this respect may be included in related SCA Attestation Rulebooks._

The SCA Attestation Provider shall check the Wallet Unit's validity (validity of the WUA) when needed and invalidate issued SCA Attestations for revoked Wallet Units.

The Relying Party, after having received the presentation of an SCA Attestation, shall verify the validity of the presentation and the SCA Attestation.
If the result of this verification is negative, the Relying Party shall cease the process.

### 5.3 Inclusion in Transaction Log

For each SCA Attestation presentation transaction specified in this document (regardless of whether successful or failed), in addition to data items specified in [TS10] as for "standard" presentations (not involving transactional data), the Wallet Unit **SHALL** include in its transaction log:

- For every transaction:
  - the value of the `transaction_data.payload.transaction_id` attribute,
- Additionally, for Payment Confirmation transactions as per Section 4.3.1:
  - a key-value pair where the key is "transaction_data_types.name" (string) and the value is populated with "Payment Confirmation" (string),
  - the value of the `transaction_data.payload.payee.name` attribute,
  - the value of the `transaction_data.payload.pisp.legal_name` (if available).
- Additionally, for Login and Risk-based Authentication transactions as per Section 4.3.2:
  - a key-value pair where the key is "transaction_data_types.name" (string) and the value is populated with "Login, Risk-based Authentication" (string),
  - the value of the `transaction_data.payload.service` attribute (if available).
- Additionally, for Payment Account Information Access transactions as per Section 4.3.3: 
  - a key-value pair where the key is "transaction_data_types.name" (string) and the value is populated with "Payment Account Information Access" (string),
  - the value of the `transaction_data.payload.aisp.legal_name` attribute (if available).
- Additionally, for E-mandate transactions as per Section 4.3.4:
  - a key-value pair where the key is "transaction_data_types.name" (string) and the value is populated with "E-mandate" (string),
  - the value of the `transaction_data.payload.payment_payload.payee.name` attribute (if available),
  - the value of the `transaction_data.payload.payment_payload.pisp.legal_name` attribute (if available).


## 6 References

| Reference      | Description |
|----------------|-------------|
| [eIDAS] | [Regulation (EU) No 910/2014 of the European Parliament and of the Council of 23 July 2014 on electronic identification and trust services [...], amended by Regulation (EU) 2024/1183](https://eur-lex.europa.eu/eli/reg/2014/910/) |
| [PSD2] | [Directive (EU) 2015/2366 of the European Parliament and of the Council of 25 November 2015 on payment services in the internal market [...]](https://eur-lex.europa.eu/eli/dir/2015/2366/) |
| [PSD2-RTS] | [Commission Delegated Regulation (EU) 2018/389 of 27 November 2017 supplementing Directive (EU) 2015/2366 [...] with regard to regulatory technical standards for strong customer authentication and common and secure open standards of communication](https://eur-lex.europa.eu/eli/reg_del/2018/389/) |
| [ARF] | [Architecture and Reference Framework](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/architecture-and-reference-framework-main.md) |
| [SD-JWT-VC] | [SD-JWT-based Verifiable Credentials (SD-JWT VC)](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/9) |
| [OID4VP] | [OpenID for Verifiable Presentations 1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/2) |
| [OID4VCI] | [OpenID for Verifiable Credential Issuance 1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3)
| [HAIP] | [OIDF OpenID4VC High Assurance Interoperability Profile 1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/4) |
| [JAR] | [RFC 9101 - The OAuth 2.0 Authorization Framework: JWT-Secured Authorization Request](https://www.rfc-editor.org/rfc/rfc9101.html) |
| [JSON Schema] | [JSON Schema Validation](https://json-schema.org) |
| [RFC5646] | [RFC 5646 - Tags for Identifying Languages](https://www.rfc-editor.org/rfc/rfc5646.txt) |
| [RFC7519] | [RFC 7519 - JSON Web Token (JWT)](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/312) |
| [TS10] | [Technical Specification 10: Data Portability and Download (Export)](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts10-data-portability-and-download-(export).md) |
| [ISO8601] | [ISO 8601 - Date and time format](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/405) |
| [RFC2397] | [RFC 2397 - The "data" URL scheme](https://www.rfc-editor.org/rfc/rfc2397.html) |
| [ISO4217] | [ISO 4217 - Currency codes](https://www.iso.org/iso-4217-currency-codes.html) |
| [ISO20022] | [ISO 20022 - Financial services - Universal financial industry message scheme](https://www.iso20022.org/) |
| [TS3] | [Technical Specification 3: Specification of Wallet Unit Attestations (WUA) used in issuance of PID and Attestations](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts3-wallet-unit-attestation.md) |
