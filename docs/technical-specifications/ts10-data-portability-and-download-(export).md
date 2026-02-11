
<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Data Portability and Download (Export)

## Abstract

The present document specifies the common format and data set for the transaction log and the Migration Object, as well as protocols related to exporting those objects from a Wallet Unit and importing to another one in a backup, recovery and migration scenarios, as required by the [European Digital Identity Regulation (EU 2024/1183)](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex%3A32024R1183) and [CIR 2024/2979](https://eur-lex.europa.eu/eli/reg_impl/2024/2979/oj/eng).

### [GitHub discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/discussions/407)

## Licensing and Reuse

© European Union, 2025-2026.

This document is made available under the Creative Commons Attribution 4.0 International licence (CC BY 4.0), unless otherwise stated.

You may reuse this document provided that appropriate credit is given and any changes are indicated.

The full licence text is available at:
https://creativecommons.org/licenses/by/4.0/

## Versioning

| Version | Date        | Description                              |
|---------|-------------|------------------------------------------|
| `0.1`   | 2025-06-27  | Initial version for the first discussion |
| `0.2`   | 2025-07-14  | Update after the first focus meeting     |
| `0.3`   | 2025-07-24  | Update after the second focus meeting    |
| `0.4`   | 2025-08-05  | Update after the public release of the draft     |
| `0.9`   | 2025-08-07  | Release candidate      |
| `0.10`  | 2025-08-14  | Release candidate - adding a general migration process description, basing on the commenting table proposal       |
| `0.11`  | 2025-08-15  | Final draft for review      |
| `1.0`   | 2025-08-15  | Final version for release      |
| `1.1`   | 2026-02-02  | Alignment with [TS12], adding non-device bound attestations handling, rework of pseudonym related aspects       |
| `1.1.1` | 2026-02-11  | Editorial update (licensing and reuse clarification) |

## 1 Introduction

### 1.1 Overview

According to [Regulation (EU) No 910/2014] with the amending [Regulation (EU) 2024/1183] (hereinafter together referred as the [European Digital Identity Regulation]) and [CIR on Integrity and Core Functionalities], the Wallet Units SHALL keep a transaction log, as well as a "migration object" that can be used to export User's data from a Wallet Unit and to import it to another Wallet Unit, in backup, recovery and migration processes.
The relevant requirements are set out in Article 5a(4)(f) and (g) of the [European Digital Identity Regulation] and Article 9 and 13 of the [CIR on Integrity and Core Functionalities].

The migration process has the following steps:

- creation of the migration object (which is a password-protected and encrypted file) in the existing Wallet Unit, which contains the list of all credentials (PIDs and Attestations), all non-device bound credentials and the transaction log,
- exporting and storing the migration object in a User preferred location, such as an external disk or a cloud storage (from locations supported by the Wallet Solution),
- importing the migration object to a new Wallet Unit,
- initiating the process of migration i.a. with use of the password to decrypt the file,  
- (automatic) issuance to the new Wallet Unit of PID and all device-bound Attestations that were present in the old Wallet Unit (as listed in the migration object's list of credentials),
- copying to the new Wallet Unit all non-device bound attestation,
- restoring in the new Wallet Unit the transaction log (from the migration object's transaction log).

Further details and requirements related to the migration process are presented in the ARF Annex 2 Topic 34.

### 1.2 Scope of the document

The present document specifies:

- the common format and content for the transaction log,
- the common format and content for the migration object,
- security measures to protect confidentiality, integrity and authenticity of the transaction log and the migration object, in storage, backup, recovery and migration processes.

The migration object contains:

- a file containing the list of credentials (PIDs and Attestations) present in a Wallet Unit,
- all non-device bound attestations,
- a transaction log file.

### 1.3 Requirements Notation

The key words "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC2119 RFC8174 when, and only when, they are written in all capital letters.

## 2 Data Model

The present data model has been created based on the legal requirements of [European Digital Identity Regulation] (including the related implementing acts),  and [ARF], while keeping the [SEMIC Style Guide](https://semiceu.github.io/style-guide/1.0.0/index.html) in mind and aligning it with the [Core Business Vocabulary](https://semiceu.github.io/Core-Business-Vocabulary/releases/2.2.0/) as much as reasonable.

![MigrationObjectAndTransactionLogDataModel](img/ts10-eudiw-mig-obj-trans-log-datamodel.svg)

As outlined in the above figure, it contains the following **main classes**:

- **Transaction**
- **Presentation**
- **W2WPresentationRequest**
- **W2WPresentation**
- **CredentialIssuance**
- **CredentialDeletion**
- **PseudonymGeneration**
- **PseudonymDeletion**
- **PseudonymousAuthentication**
- **CertificateIssuance**
- **CertificateDeletion**
- **SigningSealing**
- **DataDeletionRequest**
- **DPAReport**
- **OtherTransaction**
- **TransactionLog**
- **TransactionLogObject**
- **ListOfCredentials**
- **MigrationObject**

These classes are defined using the following **auxiliary classes**:

- **TransactionIdentifier**
- **CredentialInfo**
- **ClaimInfo**
- **Pseudonym**

These classes depend on the following auxiliary classes specified in the [Relying Party registration specification] and the [Provider information specification]:

- **Credential**
- **LegalEntity**
- **LegalPerson**
- **NaturalPerson**
- **IntendedUse**
- **Identifier**
- **MultiLangString**
- **Policy**

## 3 Specification

### 3.1 Transaction

The `Transaction` class is the central class of the transaction log data model and it assembles the generic attributes related to a transaction performed with or by a Wallet Unit, relevant for the transaction log creation. It contains the attributes specified in the following table:

| Attribute          | Multiplicity | Type  | Description  |
|--------------------|--------------|-------|--------------|
| `presentation`      | [0..1]       | [*Presentation*](#32-presentation)      | specifies the specific attributes of a **presentation** transaction. This attribute is present, if and only if the transaction is a presentation transaction.        |
| `w2wPresentationRequest`      | [0..1]       | [*W2WPresentationRequest*](#33-w2wpresentationrequest)      | specifies the specific attributes of a **presentation request** sending (in-bound) transaction.  This attribute is present, if and only if the transaction is sending a presentation request and logged in the transaction log of the requestors Wallet Unit.        |
| `w2wPresentation`      | [0..1]       | [*W2WPresentation*](#34-w2wpresentation)      | specifies the specific attributes of a **presentation** transaction performed by the Wallet Unit upon request received from another Wallet Unit, in the wallet-to-wallet interaction scenario.  This attribute is present, if and only if the transaction is a presentation transaction in wallet-to-wallet mode.        |
| `credentialIssuance`      | [0..1]       | [*CredentialIssuance*](#35-credentialissuance)      | specifies the specific attributes of a **credential (attestation) issuance** transaction attribute is present, if and only if the transaction is credential issuance or re-issuance.        |
| `credentialDeletion`      | [0..1]       | [*CredentialDeletion*](#36-credentialdeletion)      | specifies the specific attributes of a **credential deletion** transaction attribute is present, if and only if the transaction is deletion of a credential from the Wallet Unit.        |
| `pseudonymGeneration`      | [0..1]       | [*PseudonymGeneration*](#37-pseudonymgeneration)      | specifies the specific attributes of a **pseudonym generation** transaction attribute is present, if and only if the User generates a pseudonym in the Wallet Unit.        |
| `pseudonymDeletion`      | [0..1]       | [*PseudonymDeletion*](#38-pseudonymdeletion)      | specifies the specific attributes of a **pseudonym deletion** transaction. This attribute is present, if and only if the User deletes a previously generated pseudonym in the Wallet Unit.        |
| `pseudonymousAuthentication`      | [0..1]       | [*PseudonymousAuthentication*](#39-pseudonymousauthentication)      | specifies the specific attributes of a **pseudonymous authentication** transaction. This attribute is present, if and only if the User performs authentication to a Relying Party with use of a  pseudonym previously generated in the Wallet Unit .        |
| `certificateIssuance`      | [0..1]       | [*CertificateIssuance*](#310-certificateissuance)      | specifies the specific attributes of a **certificate issuance and/or key pair generation** transaction. The certificate and the associated key pair can be used for signing or sealing. This attribute is present, if and only if the transaction is about signing or sealing certificate issuance and/or associated key pair generation. This attribute is mandatory if the certificate and/or the key pair is generated, stored or processed (during the issuance/generation) inside the Wallet Unit.        |
| `certificateDeletion`      | [0..1]       | [*CertificateDeletion*](#311-certificatedeletion)      | specifies the specific attributes of a **certificate and/or key pair deletion** transaction. This attribute is present, if and only if the transaction is about (for signing or sealing) certificate and/or key pair deletion .        |
| `signingSealing`      | [0..1]       | [*SigningSealing*](#312-signingsealing)      | specifies the specific attributes of a **signing or sealing** transaction.  This attribute is present, if and only if the transaction is about signature or seal creation by the User with use of the Wallet Unit.        |
| `dataDeletionRequest`      | [0..1]       | [*DataDeletionRequest*](#313-datadeletionrequest)      | specifies the specific attributes of a **data deletion request** transaction.  This attribute is present, if and only if the transaction is about sending a request to a wallet-Relying Party to delete User's data previously obtained from the User's Wallet Unit with a presentation transaction.        |
| `dpaReport`      | [0..1]       | [*DPAReport*](#314-dpareport)      | specifies the specific attributes of a **data protection authority report** transaction.  This attribute is present, if and only if the transaction is about sending a report to a data protection authority by the User directly from the Wallet Unit.        |
| `otherTransaction`      | [0..1]       | [*OtherTransaction*](#315-othertransaction)      | specifies the specific attributes of **other** types of transaction logged in the transaction log.  This attribute is present, if and only if the transaction is none of the above mentioned transaction types.      |
| `transactionIdentifier`      | [1..1]       | *string*      | contains the **transaction identifier** of the transaction processed by the Wallet Unit.     |
| `time`      | [1..1]       | *string*      |           contains the **date and time** of the transaction in ISO 8601 format (YYYY-MM-DDTHH:mm:ss).         |
| `transactionType`      | [1..1]       | *string*     |           contains the **type** of the transaction, where the following values are defined  in the present document: <ul><li>`Presentation`</li><li>`PseudonymPresentation`</li><li>`W2WPresentation`</li><li>`W2WPresentationRequest`</li><li>`CredentialIssuance`</li><li>`CredentialReissuance`</li><li>`CredentialDeletion`</li><li>`PseudonymGeneration`</li><li>`PseudonymDeletion`</li><li>`CertificateIssuance`</li><li>`CertificateDeletion`</li><li>`SigningSealing`</li><li>`DataDeletionRequest`</li><li>`DPAReport`</li><li>`OtherTransaction`</li></ul>         |
| `transactionResult`     | [1..1]       | *string*      |         specifies the **result** of the transaction, where the following values are defined in the present document: <li>`Completed`</li><li>`NotCompleted`</li>         |

### 3.2 Presentation

The `Presentation` class is used within the definition of [`Transaction`](#31-transaction) and specifies the attributes related to the presentation requests received and processed by the Wallet Unit. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `interactingPartyIdentifier`       | [1..1]       | *[Identifier](#3209-identifier-external)*       |   may be present in order to specify one or more identifiers of the legal entity, as stated in an official record together with the identification data of that official record, if applicable.       |
| `interactingPartyType`       | [1..1]       | *string*      |          specifies the **type** of the party interacting with a Wallet Unit in a transaction, where the following values are defined in the present document: <li>`ServiceProvider`</li> (*Note: despite in the general terms the interacting parties can be of various types such as PID Providers, QEAA Providers, Signature Service Providers etc, as specified in the [CIR for Relying Party Registration] Annex I point 12, to request a presentation all of them will act (and will have to be previously registered) as "service providers" (additionally), and will always request the presentation in this role.*)              |
| `interactingPartyName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the party interacting with a Wallet Unit in a transaction. The interacting party name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity</li> as specified in [Provider information specification].   |
| `interactingPartyContact`      | [1..1]  |  Array of *string* objects      | contains an array with <li>`LegalEntity.country`</li> and at least one of the following attributes: <li>`LegalEntity.email`</li><li>`LegalEntity.phone`</li><li>`LegalEntity.infoURI`</li> as specified in [Provider information specification]. The contact details are primarily needed to enable the User to request deletion of the User's personal data obtained by the interacting party.      |  
| `isIntermediary`    | [1..1]       | *boolean*      |       indicates whether the interacting party is an **intermediary** ('TRUE') or not ('FALSE').                            |
| `intermediaryIdentifier`       | [0..1]       | *[Identifier](#3209-identifier-external)*       |   specifies one or more identifiers of the legal entity, as stated in an official record together with the identification data of that official record, if applicable.  Not present if `isIntermediary` value is 'FALSE'.      |
| `intermediaryName`      | [0..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the party interacting with a Wallet Unit in a transaction. The interacting party name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity </li> as specified in [Provider information specification]. Not present if `isIntermediary` value is 'FALSE'.   |
| `intermediaryContact`      | [0..1]  |  Array of *string* objects      | may contain an array with the following attributes: <li>`LegalEntity.country`</li><li>`LegalEntity.email`</li><li>`LegalEntity.phone`</li><li>`LegalEntity.infoURI`</li> as specified in [Provider information specification]. This attribute is present if such contact details are available. Not present if `isIntermediary` value is 'FALSE'.      |  
| `registrarURL`    | [1..1]       | *string*      |       specifies a Unique Resource Locator (URL) with a **link to the registrar's on-line service**, where the registration information about the interacting party can be retrieved from.  If `isIntermediary` value is 'TRUE', this attribute refers to the intermediated Wallet-Relying Party.                           |
| `purpose`       | [1..*]       |   [*MultiLangString*](#32010-multilangstring-external)    |        contains an array with one or more purposes of the intended data processing according to Article 5 1. (b) of (EU) 2016/679 as `IntendedUse.purpose` attribute specified in [Relying Party registration specification]. If `isIntermediary` value is 'TRUE', this attribute refers to the intermediated Wallet-Relying Party.                   |
| `privacyPolicy`    | [1..*]       | *[Policy](#32011-policy-external)* |     specifies one or more purposes of the intended data processing according to Article 5 1. (b) of (EU) 2016/679, and equals to `IntendedUse.privacyPolicy` attribute specified in [Provider information specification], where `Policy.type` SHALL be of type Privacy Statement. If `isIntermediary` value is 'TRUE', this attribute refers to the intermediated Wallet-Relying Party.            |
| `dpaName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     specifies **name** of the competent data protection authority. If `isIntermediary` value is 'TRUE', this attribute refers to the intermediated Wallet-Relying Party.  |
| `dpaCountry`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     specifies **country** of the competent data protection authority. If `isIntermediary` value is 'TRUE', this attribute refers to the intermediated Wallet-Relying Party.|
| `dpaContact`      | [1..1]  |  Array of *string* objects      | contains an array with at least one of the contact details to the competent data protection authority, where the following attributes are possible: <li>`LegalEntity.email`</li><li>`LegalEntity.phone`</li><li>`LegalEntity.infoURI`</li> as specified in [Provider information specification]. `LegalEntity.infoURI` refers here an URL to a  webform for sending the report.  If `isIntermediary` value is 'TRUE', this attribute refers to the intermediated Wallet-Relying Party.     |  
| `listOfClaimsRequested`      | [1..*]       | Array of [*ClaimInfo*](#3202-claiminfo) objects      | contains an array with information about all the claims (Wallet Unit User-related attributes) requested from the User's Wallet Unit by an interacting party.  |
| `listOfClaimsPresented`      | [1..*]       | Array of [*ClaimInfo*](#3202-claiminfo) objects      | contains an array with information about all the claims (User-related attributes) presented by the User's Wallet Unit to an interacting party. (_Note: the User may choose to release only some of the requested attributes._)     |
| `transactionalData`    | [0..1]       | [*TransactionalData*](#32012-transactionaldata)      |       contains **transactional data** transported in the presentation request, as specified in the attestation-related technical specification or rulebook; included if and only if the attestation type supports transactional data and where the transactional data was contained in the presentation request.              |
| `reasonOfNoncompletion`    | [0..1]       | *string*      |       specifies the **reason of non completion** of a transaction; included if and only if `TransactionResult` value is `NotCompleted`.              |



### 3.3 W2WPresentationRequest

The `W2WPresentationRequest` class is used within the definition of [`Transaction`](#31-transaction) for presentation requests sent out from the Wallet Unit directly to another one in a wallet-to-wallet scenario. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `listOfClaimsRequested`      | [1..*]       | Array of [*ClaimInfo*](#3202-claiminfo) objects      | contains an array with information about all the **claims (attributes) requested** from another Wallet Unit in the wallet-to-wallet presentation request.     |
| `listOfClaimsPresented`      | [1..*]       | Array of [*ClaimInfo*](#3202-claiminfo) objects      | contains an array with information about all the **claims (attributes) presented** to the Wallet Unit following the wallet-to-wallet presentation request.|

### 3.4 W2WPresentation

The `W2WPresentation` class is used within the definition of [`Transaction`](#31-transaction) for presentation requests processed by the Wallet Unit received directly from another Wallet Unit in a wallet-to-wallet scenario. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `listOfClaimsRequested`      | [1..*]       | Array of [*ClaimInfo*](#3202-claiminfo) objects      | contains an array with information about all the **claims (attributes) requested** by requested by another Wallet Unit in the wallet-to-wallet presentation request.     |
| `listOfClaimsPresented`      | [1..*]       | Array of [*ClaimInfo*](#3202-claiminfo) objects      | contains an array with information about all the **claims (attributes) presented** to another Wallet Unit following the wallet-to-wallet presentation request.|
| `reasonOfNoncompletion`    | [0..1]       | *string*      |       specifies the **reason of non completion** of a transaction; included if and only if `TransactionResult` value is `NotCompleted`.                            |

### 3.5 CredentialIssuance

The `CredentialIssuance` class is used within the definition of [`Transaction`](#31-transaction) and specifies the attributes related to credential (PID and attestation) issuance transactions. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `interactingPartyIdentifier`       | [1..1]       | *[Identifier](#3209-identifier-external)*       |   specifies one or more identifiers of the legal entity, as stated in an official record together with the identification data of that official record, if applicable.       |
| `interactingPartyType`       | [1..1]       | *string*      |          specifies the **type** of the party interacting with a Wallet Unit in the credential issuance transaction, where the following values are defined in the present document (following [CIR for Relying Party Registration] Annex I point 12): <li>`QEAAProvider`</li><li>`NonQEAAProvider`</li><li>`PubEEAProvider`</li><li>`PIDProvider`</li>             |
| `interactingPartyName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the issuer of the credentials in the issuance transaction. The issuer's name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity</li> as specified in [Provider information specification]. (*Note: In some Member States the name of the legal person may be given in more than one language, which necessitates use of *[MultiLangString](#32010-multilangstring-external)*  type of the class*).   |
| `interactingPartyContact`      | [1..1]  |  Array of *string* objects      | contains an array with <li>`LegalEntity.country`</li> and at least one of the following attributes: <li>`LegalEntity.email`</li><li>`LegalEntity.phone`</li><li>`LegalEntity.infoURI`</li> as specified in [Provider information specification].     |  
| `credentialNumberRequested`      | [1..1]       | *integer*      |      contains the total number of the credentials requested for issuance.   |
| `credentialNumberIssued`      | [1..1]       | *integer*      |      contains the total number of the credentials issued in the transaction.   |
| `credentialIdentifier`      | [1..*]       | Array of *string* objects       |    contains an array of **identifiers of the credentials** issued to a Wallet Unit in the transaction, where the identifiers values are equal to Verifiable Credential Type (`vct`) parameter value for [SD-JWT VC] format or Document Type (`docType`) parameter value for [ISO/IEC 18013-5] format.   |
| `isUserTriggered`    | [1..1]       | *boolean*      |       indicates whether the issuance transaction is triggered by the Wallet Unit User ('TRUE') or by the credential issuer ('FALSE').                            |

### 3.6 CredentialDeletion

The `CredentialDeletion` class is used within the definition of [`Transaction`](#31-transaction) for credential (PID or Attestation) deletion transactions processed by the Wallet Unit. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `credentialIdentifier`      | [1..1]       | *string*       |    contains the **identifier of the credential** deleted by the User from the Wallet Unit in the deletion transaction, where the identifier values is equal to Verifiable Credential Type (`vct`) parameter value for [SD-JWT VC] format or Document Type (`docType`) parameter value for [ISO/IEC 18013-5] format.  |
| `credentialIssuerIdentifier`       | [1..1]       | *[Identifier](#3209-identifier-external)*     |   specifies one or more identifiers of the legal entity, as stated in an official record together with the identification data of that official record, if applicable.       |
| `credentialIssuerName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the deleted credential's issuer name. The issuer's name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity</li> as specified in [Provider information specification].   |

### 3.7 PseudonymGeneration

The `PseudonymGeneration` class is used within the definition of [`Transaction`](#31-transaction) for (User's) pseudonym generation  transactions. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `pseudonym`      | [1..1]       | *[Pseudonym](#3203-pseudonym)*       |    contains the **pseudonym** of the User.   |

### 3.8 PseudonymDeletion

The `PseudonymDeletion` class is used within the definition of [`Transaction`](#31-transaction) for (User's) pseudonym deletion  transactions. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `pseudonym`      | [1..1]       | *[Pseudonym](#3203-pseudonym)*       |    contains the **pseudonym** of the User.   |


### 3.9 PseudonymousAuthentication

The `PseudonymousAuthentication` class is used within the definition of [`Transaction`](#31-transaction) and specifies the attributes related to the User authentication to a Relaying Party, where the User's pseudonym is used, the User's identity is not revealed and no any other User attributes are presented. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `interactingPartyIdentifier`       | [1..1]       | *[Identifier](#3209-identifier-external)*       |   specifies one or more identifiers of the legal entity, as stated in an official record together with the identification data of that official record, if applicable.       |
| `interactingPartyType`       | [1..1]       | *string*      |          specifies the **type** of the party interacting with a Wallet Unit in a transaction, where the following values are defined in the present document: <li>`ServiceProvider`</li><li>`NaturalPerson`</li>               |
| `interactingPartyName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the party interacting with a Wallet Unit in a transaction. The interacting party name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity</li> as specified in [Provider information specification].   |
| `pseudonym`      | [1..1]       | *[Pseudonym](#3203-pseudonym)*       |    contains the **pseudonym** of the User.   |
| `reasonOfNoncompletion`    | [0..1]       | *string*      |       specifies the **reason of non completion** of a transaction; included if and only if `TransactionResult` value is `NotCompleted`.                            |



### 3.10 CertificateIssuance

The `CertificateIssuance` class is used within the definition of [`Transaction`](#31-transaction) for signing or sealing certificate  issuance and/or associated key pair generation. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `interactingPartyIdentifier`       | [1..1]       | *[Identifier](#3209-identifier-external)*     |   specifies one or more identifiers of the legal entity, as stated in an official record together with the identification data of that official record, if applicable.       |
| `interactingPartyType`       | [1..1]       | *string*      |          specifies the **type** of the party interacting with a Wallet Unit in the certificate issuance transaction, where the following values are defined in the present document (following [CIR for Relying Party Registration] Annex I point 12): <li>`QCertForESealProvider`</li><li>`QCertForESigProvider`</li>             |
| `interactingPartyName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the party interacting with a Wallet Unit in a transaction. The interacting party name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity</li> as specified in [Provider information specification].   |
| `interactingPartyContact`      | [1..1]  |  Array of *string* objects      | contains an array with <li>`LegalEntity.country`</li> and at least one of the following attributes: <li>`LegalEntity.email`</li><li>`LegalEntity.phone`</li><li>`LegalEntity.infoURI`</li>, as specified in [Provider information specification].     |  
| `certificateIdentifier`      | [1..1]       | *string*      |     contains **identifier of the certificate** issued to the Wallet Unit in the issuance transaction, where its value is equal to `serialNumber` field value of the certificate as specified in [RFC 5280].   |

### 3.11 CertificateDeletion

The `CertificateDeletion` class is used within the definition of [`Transaction`](#31-transaction) for signing or sealing certificate deletion transactions. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `certificateIdentifier`      | [1..1]       | *string*      |     contains **identifier of the certificate** that was deleted in the transaction, where its value is equal to `serialNumber` field value of the certificate as specified in [RFC 5280].    |
| `certificateIssuerIdentifier`       | [1..1]       | *[Identifier](#3209-identifier-external)*     |   specifies one or more identifiers of the legal entity, as stated in an official record together with the identification data of that official record, if applicable.       |
| `certificateIssuerName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the party interacting with a Wallet Unit in a transaction. The interacting party name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity</li> as specified in [Provider information specification].   |

### 3.12 SigningSealing

The `SigningSealing` class is used within the definition of the [`Transaction`](#31-transaction) for signing or sealing transactions processed by the Wallet Unit. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `interactingPartyIdentifier`       | [1..1]       | *[Identifier](#3209-identifier-external)*      |   specifies one or more identifiers of the legal entity, as stated in an official record together with the identification data of that official record, if applicable.      |
| `interactingPartyType`       | [1..1]       | *string*      |          specifies the **type** of the party interacting with a Wallet Unit in a signing or sealing transaction, where the following values are defined in the present document (following [CIR for Relying Party Registration]): <li>`ESigESealCreationProvider`</li>              |
| `interactingPartyName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the party interacting with a Wallet Unit in a transaction. The interacting party name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity </li> as specified in [Provider information specification].   |
| `interactingPartyContact`      | [1..1]  |  Array of *string* objects      | contains an array with <li>`LegalEntity.country`</li> and at least one of the following attributes: <li>`LegalEntity.email`</li><li>`LegalEntity.phone`</li><li>`LegalEntity.infoURI`</li>, as specified in [Provider information specification].     |   
| `signingTransactionIdentifier`      | [1..1]       | *string*      |     specifies the **identifier of the signing or sealing** transaction processed by the Wallet Unit.   |
| `certificateIdentifier`      | [1..1]       | *string*      |     contains **identifier of the certificate** used in the signing or sealing transaction processed by the Wallet Unit, where its value is equal to `serialNumber` field value of the certificate as specified in [RFC 5280].    |
| `dtbsr`      | [1..1]       | *string*      |     contains **data to be signed representation** (DTBS/R) used in the signing or sealing transaction processed by the Wallet Unit.    |
| `fileIdentifier`      | [0..1]       | *string*      |     may contain the **identifier of the signed or sealed file**  in the signing or sealing transaction processed by the Wallet Unit, associated with `dtbsr`. This attribute is optional.   |
| `fileName`      | [0..1]       | *string*      |     may contain the **name of the signed or sealed file**  in the signing or sealing transaction processed by the Wallet Unit, associated with `dtbsr`. This attribute is optional.     |
| `fileSize`      | [0..1]       | *string*      |     may contain the **size of the signed or sealed file** signed or sealed in the signing or sealing transaction processed by the Wallet Unit, associated with `dtbsr`. This attribute is optional.     |

### 3.13 DataDeletionRequest

The `DataDeletionRequest` class is used within the definition of the [`Transaction`](#31-transaction) for sending (or sending process initiation) of the User's data deletion requests to a Wallet-Relying Party (that previously requested a presentation and obtained User's data) from the Wallet Unit. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `interactingPartyIdentifier`       | [1..1]       | *[Identifier](#3209-identifier-external)*      |   specifies one or more identifiers of the legal entity (being a Wallet-Relying Party), as stated in an official record together with the identification data of that official record, if applicable.      |
| `interactingPartyName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the interacting party, being a Wallet-Relying Party and a subject of the data deletion request. The interacting party name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity </li> as specified in [Provider information specification].   |
| `listOfClaims`      | [1..*]       | Array of [*ClaimInfo*](#3202-claiminfo) objects      | contains an array with information about all the claims (User-related attributes) in possession of an interacting party (a Wallet-Relying Party) and to be deleted by this party.    |

### 3.14 DPAReport

The `DPAReport` class is used within the definition of the [`Transaction`](#31-transaction) for sending (or sending process initiation)  of the report to a data protection authority by the Wallet Unit's User from the Wallet Unit. It contains attributes specified in the following table:

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `dpaName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     specifies **name** of the data protection authority to which the report was sent.   |
| `dpaCountry`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     specifies **country** of the data protection authority to which the report was sent. |


### 3.15 OtherTransaction

The `OtherTransaction` class is used within the definition of the [`Transaction`](#31-transaction) for any other transactions processed by the Wallet Unit, not specified earlier in the present document (for instance User's data backup or the migration object export). It contains attributes specified in the following table, as well as it may contain additional, proprietary attributes, as specified by the Wallet Solution provider (out of the scope of this document).

| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `description`      | [1..*]       | *string*      |       contains the human-readable **description** of the transaction such as type, nature, context etc. The content and syntax are at the full discretion of the Wallet Solution provider.    |

### 3.16 TransactionLog

The `TransactionLog` object is an array containing zero or more `Transaction` objects. In some cases it may be empty (if no transactions have been done yet or the log was erased).


### 3.17 TransactionLogObject

The `TransactionLogObject` object is file that contains the `TransactionLog` object. The structure and coding of `TransactionLogObject` as the exportable file is specified in Section 4.1.


### 3.18 ListOfCredentials

The `ListOfCredentials` object is a file that contains zero or more `CredentialInfo` objects. In some cases it may be empty (if no attestations are present in the Wallet Unit).


### 3.19 MigrationObject

The `MigrationObject` object is a file, that contains: the transaction log and the list of credentials (attesatation). The structure and coding of `MigrationObject` as the  exportable file is specified in Section 4.2.


| Attribute          | Multiplicity | Type | Description |
|--------------------|--------------|------|-------------|
| `transactionLog`      | [1..1]       | [TransactionLog](#316-transactionlog)      |       contains the current transaction log of the Wallet Unit.    |
| `listOfCredentials`      | [1..1]       | [ListOfCredentials](#318-listofcredentials)      |       contains the current list of all device-bound credentials (attestations) present in the Wallet Unit.    |
| 'nonDeviceBoundCredentials' | [1..1] | Array of attestation objects | Contains an array including all non-device-bound attestations currently present in the Wallet Unit; format of each entry in the array SHALL be identical to 'credential' object as received by the Wallet Unit from a Credential Issuer in a Credential Response according to section 8.3 of [OpenID4VCI]. The Wallet Unit SHALL NOT include any 'credential' objects representing device-bound credentials in the array. |


### 3.20 Auxiliary classes

#### 3.20.1 CredentialInfo

The `CredentialInfo` class represents the set of data related to a credential, that is necessary to create `ListOfCredentials` in the `MigrationObject` object. It inherits all attributes from the (external) `Credential` class. In addition it contains the attributes specified in the following table:

| Attribute                 | Multiplicity | Type | Description |
|---------------------------|--------------|------|-------------|
| `credentialIdentifier`    | [1..1]       | *string*     | contains  an **identifier of the credential** (attestation), where its value equals to Verifiable Credential Type (`vct`) parameter value for [SD-JWT VC] format or Document Type (`docType`) parameter value for [ISO/IEC 18013-5] format.  |
| `format`                  | [1..1]       | *string*               | specifies the format of the attestation, where the following values are defined in the present document:  <li>`jwt_vc_json` for the format conformant to W3C VCDM 1.1, signed as a JWT, not using JSON-LD 							(according to Appendix B.1.3.1 of [OpenID4VP])</li><li>`ldp_vc` for the format conformant to W3C VCDM 1.1, that is secured using Data Integrity, using JSON-LD (according to Appendix B.1.3.2 of [OpenID4VP])</li><li>`mso_mdoc`for the format conformant to mdoc format i.e. complying with [ISO/IEC 18013-5]												(according to Appendix B.2 of [OpenID4VP]) </li><li>`dc+sd-jwt` for the format conformant to [SD-JWT VC]	(according to Appendix B.3.1 of [OpenID4VP])</li> |
| `issuerIdentifier`       | [1..*]       | *[Identifier](#3209-identifier-external)*      |   specifies one or more identifiers of the legal entity, as stated in an official record together with the identification data of that official record, if applicable.       |
| `issuerType`       | [1..1]       | *string*      |          specifies the **type** of the issuer, where the following values are defined in the present document (following [CIR for Relying Party Registration] Annex I point 12, plus additional ones): <li>`QEAAProvider`</li><li>`NonQEAAProvider`</li><li>`PubEEAProvider`</li><li>`PIDProvider`</li>             |
| `issuerName`      | [1..1]       | *[MultiLangString](#32010-multilangstring-external)*      |     contains the **name** of the issuer. The issuer name may be: <li>`LegalPerson.legalName` for the legal person type of the legal entity, or </li><li>`NaturalPerson.givenName` + `NaturalPerson.familyName` for the natural person type of the legal entity</li> as specified in [Provider information specification].    |
| `supplyPointURL`      | [1..1]       | *string*      |     specifies a Unique Resource Locator (URL) with a **link to the issuer's on-line service**, at which a Wallet Unit can start the process of requesting and obtaining a PID or Attestation.    |


#### 3.20.2 ClaimInfo

The `ClaimInfo` class represents the set of data related to a claim (a User-related attribute). It contains the attributes specified in the following table:

| Attribute                 | Multiplicity | Type | Description |
|---------------------------|--------------|------|-------------|
| `credentialIdentifier`    | [1..1]       | *string*     | contains  an **identifier of the credential** (attestation),  where its value equals to Verifiable Credential Type (`vct`) value for [SD-JWT VC] format or Document Type (`docType`) value for [ISO/IEC 18013-5] format.  |
| `claims`    | [1..*]       | Array of *string* objects     | contains an array of path pointers that specifies the paths to the claims within the credential (see [OpenID4VP] Sections 7). |

#### 3.20.3 Pseudonym

The `Pseudonym` class represents the set of data related to User's pseudonym. It contains the attributes specified in the following table:

| Attribute                 | Multiplicity | Type | Description |
|---------------------------|--------------|------|-------------|
| `value`      | [1..1]       | *string*       |    contains the **value** of the pseudonym, which may be a public key of the asymmetric key pair generated by the Wallet Unit for this purpose.    |
| `alias`      | [0..1]       | *string*       |    may contain the User-friendly **alias**  of the pseudonym, set by the User. This attribute is optional.  |

#### 3.20.4 Credential (external)

The `Credential` class contains the attributes specified in [Relying Party registration specification].

#### 3.20.5 LegalEntity (external)

The `LegalEntity` class contains the attributes specified in [Provider information specification].

#### 3.20.6 LegalPerson (external)

The `LegalPerson` class is used within the definition of the LegalEntity class and allows to specify the attributes of a legal person. It contains the attributes specified in the [Provider information specification].

#### 3.20.7 NaturalPerson (external)

The `NaturalPerson` class is used within the definition of the LegalEntity class and allows to specify the attributes of a natural person (acting as a legal entity). It contains the attributes specified in the [Provider information specification].

#### 3.20.8 IntendedUse (external)

The `IntendedUse` class contains the attributes specified in the [Relying Party registration specification].

#### 3.20.9 Identifier (external)

The `Identifier` class contains the attributes specified in [Provider information specification].

#### 3.20.10 MultiLangString (external)

The `MultiLangString` class contains the attributes specified in the [Relying Party registration specification].

#### 3.20.11 Policy (external)

The `Policy` class contains the attributes specified in the [Provider information specification].

#### 3.20.12 TransactionalData 

The `TransactionalData` class represents data of an external transaction (such as payment authentication or signature activation), that is contained within a presentation request (for instance in `transaction_data` parameter as specified in  [OID4VP]). It contains one or more attributes specified in a technical specification or rulebook related to the attestation type at stake.

_Note: For instance, for payment authentication `TransactionalData` content is specified in [TS12]._


### 4 Structures

#### 4.1 Transaction Log Object Structure

The `TransactionLogObject` exportable object is composed from the transaction log entries of various types. It can be exported from the Wallet Unit and stored in an external location. It contains an array with transaction log entries. A single transaction log entry is a JSON-encoded `Transaction` object with key/value elements of the corresponding data types for a given transaction type, as defined in Sections 3.2 to 3.15. 

The `TransactionLogObject` object SHALL be in [JWE] format; the corresponding [JWE] ciphertext object SHALL be encoded as a [JSON] object. The encryption process and mandatory algorithms for [JWE] encryption are defined in Section 5.
  

The following is a non-normative example of the `TransactionLogObject` object:

```
{
  "transactionLogObject": [  
    {
      "interactingPartyIdentifier": {
          "type": "http://data.europa.eu/eudi/id/EUID",
          "identifier": "PLKRS.0000123456"
      },
      "interactingPartyType": "ServiceProvider",
      "interactingPartyName": "ABC Services",
      "interactingPartyContact": ["PL", "info@serviceprovider.com","0048221234567","https://www.serviceprovider.com/info/"],
      "isIntermediary": "FALSE",
      "registrarURL": "https://www.registrar.pl/registrationinfo/serviceprovider/",
      "purpose": [
        {
          "lang": "pl-PL",
          "content": "Gromadzimy Twoje imię, nazwisko, adres oraz dane europejskiej karty zdrowia, aby móc zrealizować usługę zgodnie z Twoim zamówieniem"
        }
      ],
      "privacyPolicy": {
        "type": "http://data.europa.eu/eudi/policy/privacy-policy",
        "policyURI": "https://www.serviceprovider.com/policy/"
        },
      "dpaName": "Urząd Ochrony Danych Osobowych",
      "dpaCountry": "PL",
      "dpaContact": ["kancelaria@uodo.gov.pl"],
      "listOfClaimsRequested": [
        {
          "credentialIdentifier": "urn:eudi:pid:de:1",
          "claims":["name","address"]
        },
        {
          "credentialIdentifier": "urn:eu.europa.ec.eudi:ehic:1",
          "claims":["credential_holder","starting_date","ending_date","document_id"]
        }
      ],
     "listOfClaimsPresented": [
        {
          "credentialIdentifier": "urn:eudi:pid:de:1",
          "claims":["name"]
        },
        {
          "credentialIdentifier": "urn:eu.europa.ec.eudi:ehic:1",
          "claims":["starting_date","ending_date"]
        }
      ],
      "transactionIdentifier": "346354209358604",
      "time": "2025-07-29T09:11:20",
      "transactionType": "Presentation",
      "transactionResult": "Completed"    
    },
    {
      "interactingPartyIdentifier": {
        "type": "http://data.europa.eu/eudi/id/LEI",
        "identifier": "111100AAAA9999999955"
        },
      "interactingPartyType": "ServiceProvider",
      "interactingPartyName": "Signing Service Provider",
      "interactingPartyContact": ["FR", "info@signingservice.fr","003312345678","https://www.signingservice.fr/contact/"],
      "isIntermediary": "FALSE",
      "registrarURL": "https://www.registrar.fr/registrations/signingserviceprovider/",
      "purpose": [
        {
          "lang": "en-EN",
          "content": "We collect your name to process registration to our service"
        },
        {
          "lang": "fr-FR",
          "content": "Nous collectons votre nom pour traiter l'inscription à notre service",
        }
      ],
      "privacyPolicy": {
        "type": "http://data.europa.eu/eudi/policy/privacy-policy",
        "policyURI": "https://www.signingservice.fr/privacypolicy/"
        },
      "dpaName": "CNIL",
      "dpaCountry": "FR",
      "dpaContact": ["0153732222"],
     "listOfClaimsRequested": [
        {
          "credentialIdentifier": "urn:eudi:pid:de:1",
          "claims":["firstname", "familyname"]
        }
      ],
      "listOfClaimsPresented": [],
      "transactionIdentifier": "0923485829345892458",
      "time": "2025-08-01T22:45:31",
      "transactionType": "Presentation",
      "transactionResult": "NotCompleted",
      "reasonOfNoncompletion": "session interrupted"
    }
  ]
}
```

#### 4.2 Migration Object Structure

The `MigrationObject` exportable object contains:
- `TransactionLogObject`
- `ListOfCredentials`,
- all non-device bound attestations files. 

It can be exported from the Wallet Unit and stored in an external location. The `TransactionLogObject` object's structure and coding are defined in Section 4.1.

The `ListOfCredentials` object is composed from the credential-related information entries. It contains an array with `CredentialInfo` objects, containing selected information on each device-bound credential (attestation) present in the Wallet Unit. A single credential entry is a JSON-encoded `CredentialInfo` object with key/value elements as defined in Section 3.20.1.  

The `MigrationObject` object SHALL be in [JWE] format; the corresponding [JWE] ciphertext is composed of the `TransactionLog` and `ListOfCredentials` objects, both encoded as [JSON] objects. The encryption process and mandatory algorithms for [JWE] encryption are defined in Section 5.

The following is a non-normative example of the `ListOfCredentials` object:

```
{
  "listOfCredentials":[
    {
      "credentialIdentifier": "urn:eudi:pid:de:1",
      "format": "jwt_vc_json",
      "issuerName": "ABC Issuer",
      "issuerIdentifier": {
        "type": "http://data.europa.eu/eudi/id/EUID",
        "identifier": "PLKRS.0000999999"
        },
      "issuerType": "PIDProvider",
      "supplyPointURL": "https://www.abcissuer.pl/supplypoint/"
    },
    {
      "credentialIdentifier": "https://credentials.superbank.com/iban_credential",
      "format": "jwt_vc_json",
      "issuerName": "Super Bank",
      "issuerIdentifier": {
        "type": "http://data.europa.eu/eudi/id/LEI",
        "identifier": "111100WWW00011122233"
        },
      "issuerType": "EAAProvider",
      "supplyPointURL": "https://www.superbank.com/supplypoint/"
    }
  ]
}
```

### 5 Encryption

The Wallet Unit SHALL use the following algorithms for [JWE] objects encryption:

* [PBES2-HS256+A128KW](https://www.rfc-editor.org/rfc/rfc7518.html#section-4.8), as specified in [JWA] and [PKCS#5] - to derive the key encryption key from the User's password/passphrase and encrypt the content encryption key,
* [A128GCM](https://www.rfc-editor.org/rfc/rfc7518.html#section-5.3), as specified in [JWA] - for the content encryption with the content encryption key.

The encryption process is visualised in the diagram below:

![EncryptionDiagram](img/TS10-encryption.svg)

### 6 References

| Reference      | Description |
|----------------|-------------|
| [Regulation (EU) No 910/2014] | [Regulation (EU) No 910/2014 of the European Parliament and of the Council of 23 July 2014 on electronic identification and trust services for electronic transactions in the internal market and repealing Directive 1999/93/EC](https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:32014R0910) |
| [Regulation (EU) 2024/1183] | [Regulation (EU) No 910/2014 of the European Parliament and of the Council of of 11 April 2024 amending Regulation (EU) No 910/2014 as regards establishing the European Digital Identity Framework](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex%3A32024R1183)  |
| [CIR on Integrity and Core Functionalities]  | [Commission Implementing Regulation (CIR) EU 2024/2979 of 28 November 2024 laying down rules for the application of Regulation (EU) No 910/2014 of the European Parliament and of the Council as regards the integrity and core functionalities of European Digital Identity Wallets](https://eur-lex.europa.eu/eli/reg_impl/2024/2979/oj/eng)
| [CIR for Relying Party Registration]      | [Commission Implementing Regulation (EU) 2025/848 of 6 May 2025 laying down rules for the application of Regulation (EU) No 910/2014 of the European Parliament and of the Council as regards the registration of wallet-relying parties](http://data.europa.eu/eli/reg_impl/2025/848/oj)  |
| [Provider information specification]      | [Specification of systems enabling the notification and subsequent publication of Provider information](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts2-notification-publication-provider-information.md) |
| [Relying Party registration specification]      | [Specification of common formats and API for Relying Party Registration information](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md) |
| [ARF]      | [Architecture and Reference Framework](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/architecture-and-reference-framework-main.md) |
| [JWE]      | [JSON Web Encryption (JWE), RFC 7516](https://datatracker.ietf.org/doc/rfc7516/) |
| [PKCS#5]      | [PKCS #5: Password-Based Cryptography Specification Version 2.1, RFC 8018](https://datatracker.ietf.org/doc/rfc8018/) |
| [JWA]      | [JSON Web Algorithms (JWA), RFC 7518](https://datatracker.ietf.org/doc/rfc7518/) |
| [JSON]      | [The JavaScript Object Notation (JSON) Data Interchange Format, RFC 8259](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/79) |
| [ECCG]      | [ECCG Agreed Cryptographic Mechanisms - version 2](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/416) |
| [SD-JWT-VC]      | [SD-JWT-based Verifiable Credentials (SD-JWT VC)](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/9)|
| [ISO/IEC 18013-5]  | [ISO/IEC 18013-5 - Mobile driving licence (mDL) application, First edition, 2021-09](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/84) |
| [OID4VP] | [OpenID for Verifiable Presentations 1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/2) |
| [OID4VCI] | [OpenID for Verifiable Credential Issuance 1.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/3) |
| [RFC 5280] | [RFC 5280 Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile](https://www.rfc-editor.org/info/rfc5280) |
| [TS12] | [Technical Specification 12: Specification of Strong Customer Authentication (SCA) Implementation with the Wallet](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts12-electronic-payments-SCA-implementation-with-wallet.md) |
