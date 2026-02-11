<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification of common formats and API for Relying Party Registration information

## Abstract

The present document specifies the data formats and application programming interface (API) for the machine-readable Relying Party Registration required by the [European Digital Identity Regulation (EU 910/2014)](https://eur-lex.europa.eu/eli/reg/2014/910/oj/eng).

### [GitHub discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/discussions/357)

## Licensing and Reuse

© European Union, 2025-2026.

This document is made available under the Creative Commons Attribution 4.0 International licence (CC BY 4.0), unless otherwise stated.

You may reuse this document provided that appropriate credit is given and any changes are indicated.

The full licence text is available at:
https://creativecommons.org/licenses/by/4.0/

## Versioning

| Version | Date        | Description                                               |
|---------|-------------|-----------------------------------------------------------|
| `0.1`   | 2025-04-25  | Initial version for first discussion                      |
| `0.2`   | 2025-04-30  | Initial version - fixed data model figure                 |
| `0.3`   | 2025-05-13  | Updated version based on first feedback round             |
| `0.4`   | 2025-05-23  | Updated version based on focus group meeting 2            |
| `1.0`   | 2025-06-13  | Published version after Coordination Group review period  |
| `1.1`   | 2025-10-14  | First maintenance release - JWS API fix, intendedUseIdentifier, registryURI, TS11-implied updates     |
| `1.2`   | 2025-11-27  | Added Intended use identifier to /wrp GET query parameters list in main specification, reference updates.       |
| `1.2.1` | 2026-01-30  | Editorial update (licensing and reuse clarification) |

## 1. Introduction and Overview

The present document specifies the common data formats and the application programming interface (API) related to **Relying Party Registration** according to [European Digital Identity Regulation (EU 910/2014)](https://eur-lex.europa.eu/eli/reg/2014/910/oj/eng) - herein [Regulation] and the Commission Implementing Regulation
(CIR) [EU 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj) of 6 May 2025 on laying down rules for the application of Regulation (EU) No 910/2014 of the European Parliament and of the Council as regards the registration of wallet-relying parties - herein [CIR for Relying Party Registration].

Relying Party Registration is addressed in Article 5b (11) of the Regulation and the [CIR for Relying Party Registration] lays down further details, which are in scope of present technical document.

Specifically on [CIR for Relying Party Registration]:

In the recitals, recital (3) states:
*To ensure broad access to the registers and to achieve interoperability, Member States should set up both human and machine-readable interfaces that meet the technical specifications set out in this Regulation. Providers of wallet-relying party access certificates and wallet-relying party registration certificates, where available, should, for the purpose of issuing those certificates, also be able to rely upon these interfaces.*

For the purposes of defining the technical specification, the [CIR for Relying Party Registration] sets specific ruling for:

1. The minimum common data set: The registration of a Relying Party to a registry shall include at least the information set out in Annex I of the [CIR for Relying Party Registration] - see Art. 3(2) and the technical specification [\[Common Set of Relying Party Information to be Registered\]](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts6-common-set-of-rp-information-to-be-registered.md).
2. Online access to the registry information: The data shall be made available online both in human-readable form and in a form suitable for automated processing - see Art. 3(4).
3. Information shall be available through a single common API - see Art. 3(5).
4. Common requirements of [CIR for Relying Party Registration] Annex II Section 2 shall be complied with - see Art 3(6).
5. Annex II Sections 1 and 2 define the data format, signing and API requirements:
    - a REST API published according to OpenAPI version 3,
    - serving data out in JSON format,
    - the output JSON data is signed according to IETF 7515 on JSON Web Signatures (JWS),
    - provide open access,
    - provide API methods for **search and request lists of wallet-relying parties** based on following query parameters: official or business registration number, company name, privacy policy URL, entitlements (including sub-entitlements if defined by a Member State), and wallet-relying party reliance on intermediary and by an associated intermediary where applicable.
    - where query results match at least one wallet-relying party, response shall provide statement information of these wallet relying parties, including information registered according to [CIR for Relying Party Registration] Annex I and full wallet-relying party access certificate (and registration certificate, if applicable) histories, but shall exclude physical address of the registration information of [CIR for Relying Party Registration] Annex I (4).

## 2. Data Model

The data model and common format for Relying Party Registration is derived from the [CIR for Relying Party Registration], the technical specification on [Common Set of Relying Party Information to be Registered](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts6-common-set-of-rp-information-to-be-registered.md) and the [Provider information specification],
while keeping the [SEMIC Style Guide](https://semiceu.github.io/style-guide/1.0.0/index.html) in mind and aligning it with the [SEMIC Core Business Vocabulary](https://semiceu.github.io/Core-Business-Vocabulary/releases/2.2.0/) as much as reasonable.

> NOTE: The data model for Relying Party Registration data depends on the [Provider information specification] data model, to which it carries a few superclass dependencies as referenced in this section.

![WRP-Reg-DataModel](img/ts5-wrpreg-datamodel.svg)

As outlined in the figure, the data model contains the main class **WalletRelyingParty**, which has class dependencies to superclasses **LegalEntity** and **Provider** specified in [Provider information specification].

The main class of the data model is **WalletRelyingParty**, which

- contains the attributes listed in Section 2.1
- as a class inherits attributes of **Provider** and **LegalEntity** superclasses from the [Provider information specification].
- defines its own **auxiliary classes**:
  - **Claim**
  - **Credential**
  - **IntendedUse**
  - **MultiLangString**

- depends on following auxiliary classes of the [Provider information specification]:
  - **Identifier**
  - **Policy**
  - **LegalPerson**
  - **NaturalPerson**
  - **Law**

The information present in the optionally issued Relying Party Registration Certificates (issuance of RPRC is one for each Intended use of the Wallet-Relying Party), specified in \[ETSI 119 475\], is gathered from these classes.

> NOTE: A mapping of the data elements in the model versus the RPRC contents will be provided once the final 1.0.0 ETSI specification is released, along with an API method to receive the RPRC contents via the Registry API.

### 2.1 WalletRelyingParty

The `WalletRelyingParty` main class inherits all attributes from the `Provider`
superclass specified in the [Provider information specification]. In addition to the attributes of the `Provider` class it contains the attributes specified in the following table:

| Attribute              | Multiplicity | Type                                   | Description     |
|------------------------|--------------|----------------------------------------|-----------------|
| `tradeName`            | [0..1]       | *string*                               | may be present in order to specify the **trade name** (common name, service name) of the Wallet-Relying Party, if applicable.           |
| `supportURI`           | [1..*]       | Array of *strings*                               | specifies the **support URI** for the service provided by the Wallet-Relying Party. Information is mandatory for the purpose of sending the Wallet-Relying Party a data deletion request according to the requirements set in the [Specification of Common Interface for Data Deletion Requests]. Note that Annex I (7) of [CIR for Relying Party Registration](https://TBA) only stipulates that the Wallet-Relying Party SHALL provide detailed contact information in form of (a) a website for helpdesk and support, (b) a phone number or (c) an e-mail address pertaining to its registration and intended use of the Wallet Units. **For the requirements specified in the [Specification of Common Interface for Data Deletion Requests] the Wallet-Relying Party is strongly recommended to register at least the website URL or the email-address for the `supportURI` attribute.** |
| `srvDescription`       | [1..*]       | Array of *[MultiLangString](#245-multilangstring) objects*                      | contains an array of arrays with localised **descriptions of the services provided by the Wallet-Relying Party**.                              |
| `intendedUse`          | [0..*]       | Array of [*IntendedUse*](#243-intendeduse) objects      | array of `IntendedUse` objects, may appear one or more times in order to specify intended use cases in which the Wallet-Relying Party intends to rely on attestations of attributes of a Wallet User presented by a Wallet Unit. `IntendedUse` is **not required** from Wallet-Relying Parties that register only to act as an intermediary. Note that the attestation providers requesting attestations of attributes against their attestation provisioning service will need to register their intended use(s) as normal service providers do.                                    |
| `isPSB`                | [1..1]       | *boolean*                              | indicates whether the Wallet-Relying Party **is a public sector body** or not. The attribute SHALL be provided and be set to `FALSE` if registered entity is not a public sector body.                       |
| `entitlement`          | [1..*]       | Array of *strings*                               | specifies the **set of entitlements** of the Wallet-Relying Party in form of a URI according to [RFC3986](https://www.rfc-editor.org/rfc/rfc3986.html). Note that a Wallet-Relying Party may register for multiple entitlements. An attestation provider that requires presentation of another attestation during issuance of their own attestation SHALL register both as a Service_Provider and with their attestation provider entitlement in a single registration. The following URIs are specified for the roles present in the [CIR for Relying Party Registration]: <ul><li> https://uri.etsi.org/19475/Entitlement/Service_Provider</li> <li> https://uri.etsi.org/19475/Entitlement/QEAA_Provider</li><li> https://uri.etsi.org/19475/Entitlement/Non_Q_EAA_Provider</li><li> https://uri.etsi.org/19475/Entitlement/PUB_EAA_Provider</li><li> https://uri.etsi.org/19475/Entitlement/PID_Provider</li><li> https://uri.etsi.org/19475/Entitlement/QCert_for_ESeal_Provider</li><li> https://uri.etsi.org/19475/Entitlement/QCert_for_ESig_Provider</li><li> https://uri.etsi.org/19475/Entitlement/rQSealCDs_Provider</li><li> https://uri.etsi.org/19475/Entitlement/rQSigCDs_Provider</li><li> https://uri.etsi.org/19475/Entitlement/ESig_ESeal_Creation_Provider</li></ul>     |
| `providesAttestations` | [0..*]       | Array of [*Credential*](#244-credential) objects     | A list of self-declared or referenced attestation types the Wallet-Relying Party intends to issue for Wallet Units. Shall be present only if any `entitlement` of the Wallet-Relying Party is of type QEAA_Provider, Non_Q_EAA_Provider, PUB_EAA_Provider or PID_Provider, listing the attestation type(s) (`Credential`) the Wallet-Relying Party intends to issue to Wallet Units. Referenceable attestation types are listed in a publicly available attestation catalogue, such as the one defined in [Specification of interfaces and formats for the catalogue of attributes and the catalogue of attestations]. The attribute to be referenced for a catalogued attestation type is its catalogue-assigned attestation schema URI (`SchemaMeta.Schema.uri`)    |
| `supervisoryAuthority` | [1..1]       | [*LegalEntity*](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts2-notification-publication-provider-information.md#21-legalentity) | specifies the competent **data protection supervisory authority** according to Article 51 of [(EU) 2016/679](http://data.europa.eu/eli/reg/2016/679/oj) in charge of supervising the Wallet-Relying Party. It provides the necessary contact information towards the Data Protection Authority through its `email`, `phone` and/or `infoURI` attributes, of which at least one SHALL be provided for an DPA registered into a Member State Registry.         |
| `registryURI`            | [1..1]       | *string*                               | URI for the national registry API of the registered Wallet-Relying Party. The national Registrar SHALL provide this information upon registration. For other Registrar-provided data, see also `intendedUseIdentifier` in the `IntendedUse` class.         |
| `usesIntermediary`     | [0..*]       | Array of [*WalletRelyingParty*](#21-walletrelyingparty) objects                 | if present, indicates **whether the Wallet-Relying Party depends on use of at least one intermediary** and lists the needed information (`WalletRelyingParty.identifier`, `WalletRelyingParty.tradeName` and `WalletRelyingParty.registryURI`) of the intermediaries the registered Wallet-Relying Party depends on. The URL to the Registrar of the intermediary is necessary as it may have different Registrar than the Wallet Relying Party depending on it. See [ARF Topic 52](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-2/annex-2-high-level-requirements.md#a2352-topic-52-relying-party-intermediaries), high-level requirements RPI_01 and RPI_07.             |
| `isIntermediary`       | [1..1]       | *boolean*                              | indicates whether the Wallet-Relying Party is registered to act on-behalf of other Relying Parties (is an intermediary or not). Attribute SHALL be set to FALSE if `usesIntermediary` attribute is present. |

> Note: Topic M (User reporting unlawful or suspicious request of data to Data Protection Authorities) of the ARF adds the contact information of intended-use responsible DPA to the set of data to be registered by the Wallet-Relying Party. This information is available as OneOf[supervisoryAuthority.email, supervisoryAuthority.phone, supervisoryAuthority.infoURI] if the same DPA is valid for all intended uses of the Wallet-Relying Party.

### 2.2 Provider

The `Provider` superclass inherits all attributes from the more general `LegalEntity` superclass specified in the [Provider information specification]. WalletRelyingParty class inherits all attributes of this class.

### 2.3 LegalEntity

The `LegalEntity` superclass contains the attributes specified in [Provider information specification]. WalletRelyingParty class inherits all attributes of this class.

### 2.4 Auxiliary Classes

The WalletRelyingParty class has attributes that depend on a set of its own auxiliary classes (in addition to ones defined in [Provider information specification] as listed in Section 2 above) as follows:

### 2.4.1 Claim

The `Claim` class is used within the definition of the [`Credential`](#244-credential) class to list required attributes within a requested attestation, and contains the following attributes:

| Attribute                 | Multiplicity | Type                   | Description                           |
|---------------------------|--------------|------------------------|---------------------------------------|
| `path`                    | [1..1]       | *non-empty array of strings, nulls and non-negative integers*               | a path pointer that specifies the path to a claim within the `Credential`. See [OpenID4VP] Sections 6.3. and 7. Section 7.1 specifies use of path for Credentials in sd-jwt-vc format, and Section 7.2 for Credentials in mso_mdoc format. |
| `values`                  | [0..1]       | *An array of strings, integers or boolean values*               | optional array of strings, integers or boolean values that specifies the expected values of the claim (used to indicate that the Wallet should return the claim only if the type and value of the claim both match exactly for at least one of the elements in the array). See [OpenID4VP] Sections 6.3. and 6.4.1.                    |

`Claim` SHALL NOT be present in [`Credential`](#244-credential) objects declared by attestation providers in the `WalletRelyingParty.providesAttestations`array. Full set of claims of an attestation schema are declared behind the attestation type and/or Attestation Rulebook URL, which SHALL be referenced in the `Credential.meta` attribute.

### 2.4.2 Identifier (external)

The `Identifier` class is used within the definition of the `LegalEntity` superclass and contains the attributes specified in the [Provider information specification].

#### 2.4.2.1 Unique Identifier for identification of Wallet-Relying Parties

The ARF Annex 2 high-level requirements on Relying Party registration and authentication - see [Topic 44](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-2/annex-2-high-level-requirements.md#a2344-topic-44---relying-party-registration-certificates), [Topic 52](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-2/annex-2-high-level-requirements.md#a2352-topic-52-relying-party-intermediaries), [Topic 6](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-2/annex-2-high-level-requirements.md#a236-topic-6---relying-party-authentication-and-user-approval) - require a Wallet Unit to be able to both identify the Wallet-Relying Party with an unique identifier and a trade name or service name of the said Relying Party, and in the optional case that Registrar issues Registration Certificates, allow association between the Access Certificate and the Registration Certificate of a Relying Party (or between an intermediary and its End-Relying Party) based on this unique identifier.

The [CIR for Relying Party Registration] Annex I (2) lists seven alternative identifiers and a wild card option (3)(h) to be registrable for a Wallet-Relying Party, of which **at least one** should be provided when registering a Wallet-Relying Party to a Registrar.

For the purpose of standardising a single technical unique identifier available for use in the EUDI framework, when available in the national business register(s), an identifier of type **`http://data.europa.eu/eudi/id/EUID` - European unique identifier (EUID)** as defined in \[[CIR for BRIS\]](https://eur-lex.europa.eu/eli/reg_impl/2021/1042/oj) SHALL be used by default in registration of a Wallet-Relying Party to a Registrar.

If EUID identifier type is not available, the Member State SHALL define and publish the national unique identifier scheme used to identify the Wallet-Relying Parties in the respective national Registry(-ies).

Other alternate identifiers listed in [CIR for Relying Party Registration] Annex I (2) MAY be registered in addition to the default unique identifier.

> Note 1: The Annex I (3)(b) 'registration number as registered in national business register' is considered redundant with EUID (3)(g) - all Member State national business registries shall provide EUIDs for all registered legal entities in their registry for cross-border purposes.
>
> Note 2: The set of identifiers in [CIR for Relying Party Registration] Annex I indicates all Wallet-Relying Parties to be legal persons. Registration of natural persons as Wallet-Relying Parties (an option enabled in the [Regulation]) can be followed utilising the `LegalEntity` class of the [Provider information specification].

#### 2.4.3 IntendedUse

The `IntendedUse` class is used within the definition of the [`WalletRelyingParty`](#21-walletrelyingparty) class to specify the required information when a Wallet-Relying Party with role of a Service Provider (see `WalletRelyingParty.entitlements`) is requesting data from a Wallet Unit. An Attestation Provider that does not intend to request any attestations from the Wallet Unit does not register an Intended use. For legal reference see [CIR for Relying Party Registration] Annex I paragraphs (8), (9) and (10). The class contains an identifier attribute assigned to the registered Intended use upon registration and is provided by the national Registrar. For Member States providing Wallet-Relying Party Registration Certificates, the local unique identifier should be the same unique identifier used as the identifier of the certificate issued.

The class contains the attributes specified in the following table:

| Attribute                 | Multiplicity | Type                   | Description                           |
|---------------------------|--------------|------------------------|---------------------------------------|
| `purpose`                 | [1..*]       | Array of [*MultiLangString*](#245-multilangstring) objects     | specifies one or more **purposes** of the intended data processing according to Article 5 1. (b) of [(EU) 2016/679](https://eur-lex.europa.eu/eli/reg/2016/679/oj). The purpose SHALL be possible to be displayed localised to the User's language (`lang`), and localisations (`content`) SHALL be provided for all official languages of Member States where the intended use is provided. The localisation SHALL be provided in accordance with Annex E of [ETSI TS119612 V2.3.1](https://www.etsi.org/deliver/etsi_ts/119600_119699/119612/02.03.01_60/ts_119612v020301p.pdf).           |
| `privacyPolicy`           | [1..*]       | Array of [*Policy*](#246-policy-external) objects | specifies the privacy policy of the intended use. `Policy.type` SHALL be of type Privacy Statement. The privacy policy SHALL be possible to be displayed localised to the User's language (locale), and localisations SHALL be provided for all official languages of Member States where the intended use is provided. The localisation, if provided through listing individual URIs for each policy language version and if intended for rendering on the EUDI Wallet Instance, SHOULD be provided in accordance with Annex E of [ETSI TS119612 V2.3.1](https://www.etsi.org/deliver/etsi_ts/119600_119699/119612/02.03.01_60/ts_119612v020301p.pdf).   *NOTE: Localisations can be arranged on the Wallet-Relying Parties with multiple approaches, using Content Management Systems (CMSs), individual web pages per language variant of the policy with User locale based redirection from the policy root page, or by using commonly available internationalisation libraries on the front-end implementation. Two last approaches would work with just one privacyPolicy URI registration.*          |
| `intendedUseIdentifier`   | [1..1]       | *string* | Registrar-provided unique identifier of the registered intended use. For Member States providing Wallet-Relying Party Registration Certificates, the local unique identifier SHOULD be the same unique identifier used as the identifier of the certificate issued.     |
| `createdAt`                   | [1..1]       | *string* | Validity start date for the intended use in [ISO8601‑1] YYYY-MM-DD format.      |
| `revokedAt`                   | [0..1]       | *string* | End date for the validity of the intended use in[ISO8601‑1] YYYY-MM-DD format. The intended use MAY be marked to expire at given date, or MAY be revoked by the Wallet-Relying Party as necessary.  Note that the [CIR for Relying Party Registration] requires expired or revoked intended use data be available beyond the validity lifespan of the intended use, during which it is used in validation of presentation requests.     |
| `credential`              | [1..*]       | Array of [*Credential*](#244-credential) objects | specifies the set of potentially **requestable attestations** which may be requested by the Wallet-Relying Party within the scope of the present intended use of data.      |

#### 2.4.4 Credential

The `Credential` class is used within the definition of the [`IntendedUse`](#243-intendeduse) class to specify the individual attestation specific information within a presentation request or upon issuance of the attestation, when registering entity is an attestation provider. The Credential class allows two complementary mechanisms to specify the attestation information: 1) linking to a URL that points to the attestation schema in a machine-readable catalogue (e.g., hosted in the European Commission hosted Catalogue of Attestations, see [Specification of interfaces and formats for the catalogue of attributes and the catalogue of attestations](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts11-interfaces-and-formats-for-catalogue-of-attributes-and-catalogue-of-schemes.md), or similar hosted by a member state or by private actor), or 2) in lack of a publicly accessible attestation schema and Attestation Rulebook, a self-declaration of the attestation schema via defining the attestation type and claims for the format.

| Attribute                 | Multiplicity | Type                   | Description                           |
|---------------------------|--------------|------------------------|---------------------------------------|
| `format`                  | [1..1]       | *string*               | Specifies the format of the attestation. For valid values, see enumerations listed in [Specification of interfaces and formats for the catalogue of attributes and the catalogue of attestations] and [OpenID4VP] Appendix B.  |
| `meta`                    | [1..1]       | *string*               | An object defining additional properties requested by the Verifier (including the credential type) that apply to the metadata and validity data of the Credential. The properties of this object are defined per Credential Format - see [OID4VP] Section 6.1 and annex [B.3.5](https://openid.net/specs/openid-4-verifiable-presentations-1_0-27.html#sd_jwt_vc_meta_parameter) for SD-JWT VC format, and annex [B2.3](https://openid.net/specs/openid-4-verifiable-presentations-1_0-27.html#mdocs_meta_parameter) for ISO mdoc format.     |
| `claim`                   | [0..*]       | [*Claim*](#241-claim)  | A non-empty array of objects that specifies attributes in the requested attestation. See [OID4VP] Section 6.3.   |

> Note: In line with the data minimisation principle according to Article 5 1. (c) of [(EU) 2016/679](https://eur-lex.europa.eu/eli/reg/2016/679/oj) the Wallet-Relying Party may only request the minimum set of attestations (`Credential`) and attributes (`Claim`) within those necessary for a specific intended use (described in `IntendedUse.purpose`).

#### 2.4.5 MultiLangString

The `MultiLangString` class is used within the definition of the [`IntendedUse`](#243-intendeduse) class to provide the country code  and string tuple in localisation of the intended use's `purpose` attribute array, and within the definition of the [`WalletRelyingParty`](#21-walletrelyingparty) class to provide the country code and string tuple in localisation of the Wallet-Relying Party's `serviceDescription` attribute array. The codes and strings shall be provided according to the rules set in Annex E of \[ETSI TS 119 612\].

| Attribute                 | Multiplicity | Type                   | Description                           |
|---------------------------|--------------|------------------------|---------------------------------------|
| `lang`                    | [1..1]       | *string*               | Specifies the country code of the localised text. |
| `content`                 | [1..1]       | *string*               | The localised text as a string.     |

#### 2.4.6 Policy (external)

The `Policy` class is used within the definition of the  [`IntendedUse`](#243-intendeduse)class. It contains the attributes specified in the [Provider information specification].

#### 2.4.7 LegalPerson (external)

The `LegalPerson` class is used within the definition of the `LegalEntity` class and allows to specify the attributes of a legal person. It contains the attributes specified in the [Provider information specification].

#### 2.4.8 NaturalPerson (external)

The `NaturalPerson` class is used within the definition of the `LegalEntity` class and allows to specify the attributes of a natural person acting as a legal entity. It contains the attributes specified in the [Provider information specification]. If an identifier of a natural person needs to be registered upon registration of a Wallet-Relying Party, the `identifier` of the `Provider` class SHALL be used for this purpose, using the `serialNumber` type as defined in [ETSI EN 319 412-1].

#### 2.4.9 Law (external)

The `Law` class is used within the definition of the `LegalPerson` class and contains the attributes specified in the [Provider information specification].

## 3 Common Application Programming Interface

### 3.1 API methods on registration and updating of Wallet-Relying Party data

#### 3.1.1 Requirements

The common Registry API write methods are defined for purposes of managing the register information of Member State Registrars, and SHALL be accessible by authorised users only.

#### 3.1.2 Method description

path:
`/wrp` (POST, PUT and DELETE):

**POST** is for creating a new WalletRelyingParty. Method expects a request body with the WalletRelyingParty schema, and returns a `201` on success.

**PUT** is for updating an existing WalletRelyingParty. Method expects a request body with the WalletRelyingParty schema, and can return `200` on success or `404` if not found.

**DELETE** is for deleting of an existing WalletRelyingParty. Method expects a request body with the WalletRelyingParty identifier, and returns a `204` on success.

> Note: **The harmonisation of the POST method (means to accomplish registration of a new Wallet-Relying Party) has been left for further study.** A basic method for creating a WalletRelyingParty instance at the Registrar is provided in the API specification in a commented-out section.

#### 3.1.3 Authentication and authorisation

The write methods SHALL be only accessible for authenticated and authorised API clients. **The national mechanisms to implement authentication and authorisation are left for the discretion of the Member States.**

The applicable mechanism SHALL be documented by adding a `securitySchemes` section in `components` and a `security` section at the root level or at the path/operation level in the individual instance of the [Annex A.2](#a2-openapi-specification-normative) specification.

> Note: Requirement to provide secure-by-design and high-availability API is especially valid for the methods listed here.

### 3.2 API methods for registrar queries (open API)

#### 3.2.1 Requirements

The Registry API's read methods SHALL be open for public access.

The public API SHALL provide methods for searching and querying complete data sets of registered Wallet-Relying Parties matching with provided query parameters, that can be any of:

- **official or business registration number** (see `WalletRelyingParty.Identifier`) - will return data of list of matching Wallet-Relying Parties
- **official company name or trade name** (see `WalletRelyingParty.legalName` and `WalletRelyingParty.tradeName`) - will return data of list of matching Wallet-Relying Parties.
- **URL of the Wallet-Relying Party's privacy policy** (see `WalletRelyingParty.policy`) - will return data of list of matching Wallet-Relying Parties.
- **type of entitlement** (see `WalletRelyingParty.entitlement`) - will return data of list of Wallet-Relying Parties matching with queried entitlement type.
- **type of attestations provided** (see `WalletRelyingParty.providesAttestations`) - - will return data of list of matching Wallet-Relying Parties that provide the queried attestation type.
- **type of attestation requested** (see `WalletRelyingParty.intendedUse`) - - will return data of list of matching Wallet-Relying Parties that request the queried attestation type within their registered intended use.
- **reliance upon an Intermediary** (see `WalletRelyingParty.usesIntermediary`) - will return data of list of matching Wallet-Relying Parties that have the attribute `WalletRelyingParty.usesIntermediary` present.
- **official company name or trade name of an Intermediary** (see `WalletRelyingParty.isIntermediary`) - will return all Wallet-Relying Parties associated with queried Intermediary information through their `WalletRelyingParty.usesIntermediary` information present in the Registry.

Where query results match at least one wallet-relying party, the JWS-signed method response SHALL provide:

- set of information registered to matching Wallet-Relying Party (the full contents of `WalletRelyingParty` class for given instance without the `WalletRelyingParty.physicalAddress` attribute), and
- complete wallet-relying party access certificate histories (the Certificate Transparency log information [RFC 9162]), as set available by respective Certificate Authorities for Access Certificates, if the Registrar is not itself acting as the respective Certificate Authority.

#### 3.2.2 Method description

paths:
`/wrp` (GET):

There is a single **GET** /wrp endpoint for making parametrised queries. For this purpose, the API specification has the `parameters` section, where:

- Each filterable field (`identifier`, `legalname`, `tradename`, `policy`, `entitlement`, `providesattestation`, `usesintermediary`, `isintermediary`, `intendeduseidentifier`, `intendedUseClaimPath`, `intendedUseCredentialMeta`, `intendedUseCredentialFormat`) is defined as a query parameter.
- `in: query`: Specifies that the parameter is a query parameter.
- `name`: The name of the query parameter.
- `schema`: Defines the data type of the query parameter (mostly `string`, with `boolean` for `isintermediary`).
- `description`: Provides a brief explanation of what the query parameter filters by.

If no query parameters are included, the method returns the full list of registered Wallet-Relying Parties in the Registry.

In addition to the basic parameter set the endpoint includes also parameters for handling cursor-based pagination, and methods to query the Registry based on Intended Use components - the four `intendedUse` filters. This parameter set is included to help Wallet Units in making validation queries towards the Registries in case the Wallet Relying Party sending a presentation request does not include (i.e. the Wallet-Relying Party has not been issued) an Relying Party Registration Certificate.

The API clients can make requests such as:

`/wrp?identifier=someIdentifier`

`/wrp?legalname=Example%20Corp`

`/wrp?policy=https://example.com/policy`

`/wrp?entitlement=http://data.europa.eu/eudi/entitlement/PUB_EAA_Provider`

`/wrp?isintermediary=true`

`/wrp?identifier=someIdentifier&claimpath=IBAN&credentialformat=dc+sd-jwt`

And even combine multiple filters: `/wrp?legalname=Another%20Org&isintermediary=false`

The JWS-signed response body for a successful GET `(200)` will be an array of matching `WalletRelyingParty` objects.

In addition to the `WalletRelyingParty` objects of matching entities, the Registrar's API SHALL provide the Certificate Transparency log information of the matching entities' Relying Party Access Certificates (RPACs) as defined in [RFC 9162]. How the Registrar arranges access to the logs from the Provider of Relying Party Access Certificates is out of scope of this specification.

`/wrp/{identifier}` (GET):

A parallel endpoint for the DELETE endpoint, a GET endpoint for requesting a singular `WalletRelyingParty` object with accompanying RPAC CT log information for an registered Wallet-Relying Party that matches the `identifier` given in the API request. Response is a signed JWT.

`/wrp/check-intended-use` (GET):

A dedicated check endpoint for allowing a highly specialised endpoint for making narrowed-down `Intended use` related queries from the Registry with one mandatory and five optional parameters (see the [API specification](api/ts5-openapi31-registrar-api.yml) for details). The query parameters of the basic `/wrp` endpoint can be used to have same outcome, but this endpoint provides a JWS-signed boolean `TRUE` or `FALSE` response, based on if the queried parameter set can be found in the Registrar's `Intended use` information for the Wallet-Relying Party.

The OpenAPI 3.1 compatible REST API methods for the above are provided in Annex A.2

#### 3.2.3 Protection against DDOS attacks and accidental overuse of the GET methods

The API implementation SHALL take into consideration the vulnerabilities that an open, comprehensive query interface enables for both malicious or non-malicious users of the interface. First well-known vulnerability is sensitivity for Distributed Denial-of-Service (DDoS) attacks, the other valid one especially for the GET/wrp endpoint is the server load that very complex query parameter combinations from badly planned or non-intended/accidental misuse of the query mechanism can cause at the Registrar's server-side solution.

##### DDoS attack protection guidelines

Following policy/recommendation SHOULD be implemented (with solution-fitted combinations) in the national Registry API implementations to protect the API from DDoS attacks:

- Use of **a Cloud-based DDoS Protection Service/CDN** as the best first line of defense for volumetric attacks.
- Use of an **API Gateway or Load Balancer with Rate Limiting** is crucial for fight against application-layer attacks targeting the API.
- Deploy a **Web Application Firewall (WAF)** to protect against other application-layer threats and to complement rate limiting.
- Ensure your infrastructure is scalable by using e.g. **Load Balancers** to handle legitimate bursts and some attack traffic.
- Utilise **Caching** heavily for the GET endpoints to reduce backend load - many responses might be static or change infrequently, and caching the responses at the CDN, load balancer or application level means the API logic and database will receive less requests.
- Have robust **API monitoring** for request rates, latency, error rates, server resource utilisation and source IP addresses with proper alerting in place.
- Utilise **network segmentation** - e.g., isolate the API servers in a private subnet behind the WAF, and from the actual Registrar database to improve robustness of your network architecture against hostile attacks.

##### Protection against unintentional misuse

The implementers SHOULD consider how to limit the load of the API from perfectly legal but unintentionally complex or frequent API calls to the query endpoints. Possible solutions for this are:

- The GET endpoints SHOULD be provided with e.g. rate limiter for calls coming in from an individual IP address.
- The GET/wrp endpoint is an example of an API method that allows throwing a lot of computational load for a server serving data from a potentially large database of Wallet-Relying Party information via requesting unnecessarily complex query parameter combinations. Best practices for limiting the number of query parameters in a valid API call SHOULD be tested with multiple Wallet Solutions - with an outcome as a documented guideline for proper use of the Registry API. Improperly configured API calls can be rejected at selected level of the backend architecture.

The Registrar SHALL publish in their respective API documentation what limits it has configured regarding rate limits for legal use, or use of the query parameters. E.g., it could be that the API allows only 3 calls per minute from each IP address, and/or queries with total of 4 parameters (both values are examples - actual limits must be tested during the API implementation and test phases).

### 4 References

| Reference                              | Description                                  |
|----------------------------------------|----------------------------------------------|
| [Regulation]                           | [European Digital Identity Regulation (EU 910/2014)](https://eur-lex.europa.eu/eli/reg/2014/910/oj/eng)  |
| [CIR for Relying Party Registration]   | [Commission Implementing Regulation (EU) 2025/848 of 6 May 2025 laying down rules for the application of Regulation (EU) No 910/2014 of the European Parliament and of the Council as regards the registration of wallet-relying parties](http://data.europa.eu/eli/reg_impl/2025/848/oj)  |
| [CIR for BRIS]                         | [CIR (EU) 2021/1042 on technical specifications and procedures for the system of interconnection of registers](https://eur-lex.europa.eu/eli/reg_impl/2021/1042/oj)      |
| [Common Set of Relying Party Information to be Registered]    |       [The European Commission Technical Specification on Common Set of Relying Party Information to be Registered](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts6-common-set-of-rp-information-to-be-registered.md)         |
| [Provider information specification]                            | [The European Commission Technical Specification of systems enabling the notification and subsequent publication of Provider information](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts2-notification-publication-provider-information.md) |
| [Specification of Common Interface for Data Deletion Requests] |  [The European Commission Specification of Common Interface for Data Deletion Requests to Relying Parties](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts7-common-interface-for-data-deletion-request.md)   |
|  [Specification of interfaces and formats for the catalogue of attributes and the catalogue of attestations]   |  [The European Commission Specification for interfaces and formats for the Catalogue of Attributes and the Catalogue of Attestations](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts11-interfaces-and-formats-for-catalogue-of-attributes-and-catalogue-of-schemes.md) |
| [ETSI TS 119 475]                      | [ETSI TS 119 475 V1.1.1 Electronic Signatures and Trust Infrastructure (ESI); Relying Party attributes supporting EUDI Wallet user's authorisation decisions](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/287)                                     |
| [ETSI TS 119 612]                      | [ETSI TS 119 612 V2.4.1 Electronic Signatures and Trust Infrastructure (ESI); Trusted Lists](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/41)                                     |
| [ETSI EN 319 412-1]                    | [ETSI EN 319 412-1 V1.6.1 Electronic Signatures and Trust Infrastructure (ESI); Certificate Profiles; Part 1: Overview and common data structures](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/143)                                     |
| [ISO8601-1]                            | [ISO 8601-1:2019 Date and Time - Representations for information interchange - Part 1: Basic rules](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/405)   |
| [RFC 9162]                             | [IETF RFC 9162 Certificate Transparency 2.0](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/404) (experimental)   |

## Annex A

### A.1 JSON Schema (normative)

The file [`ts5-json-common-rp-data-model.json`](api/ts5-json-common-rp-data-model.json) contains the JSON schema definitions corresponding to the Data Model specified in [Section 2](#2-data-model) above. It is a Wallet-Relying Party Registration -specific subset of the full data model defined in [Provider information specification].

### A.2 OpenAPI Specification (normative)

The YAML file [`ts5-openapi31-registrar-api.yml`](api/ts5-openapi31-registrar-api.yml) contains the [OpenAPI](https://spec.openapis.org/oas/latest.html) specification of the JSON and REST based application programming interfaces described in [Section 3](#3-common-application-programming-interface).

### A.3 XML Schema (informative)

The file [`ts5-xsd-common-rp-data-model.xsd`](api/ts5-xsd-common-rp-data-model.xsd) contains the XML schema definitions corresponding to the Data Model specified in [Section 2](#2-data-model).
