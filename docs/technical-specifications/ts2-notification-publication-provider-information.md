<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification of systems enabling the notification and subsequent publication of Provider information

## Abstract

The present document specifies the data model and systems enabling the notification and subsequent publication of Provider information.

### [GitHub discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/discussions/427)

## Licensing and Reuse

© European Union, 2025-2026.

This document is made available under the Creative Commons Attribution 4.0 International licence (CC BY 4.0), unless otherwise stated.

You may reuse this document provided that appropriate credit is given and any changes are indicated.

The full licence text is available at:
https://creativecommons.org/licenses/by/4.0/

## Versioning

| Version | Date       | Description                                          |
|---------|------------|------------------------------------------------------|
| `0.1`   | 2025-01-16 | Initial version for first discussions.               |
| `0.2`   | 2025-02-10 | Version with first feedback integrated.              |
| `0.3`   | 2025-03-09 | Version with more comments integrated.               |
| `0.4`   | 2025-04-09 | Version with more comments integrated and separation of ``LegalEntity`` and ``Provider``. ``IntendedUse`` has been introduced and specification text has been closely aligned with class diagram.      |
| `0.9`   | 2025-04-25 | Version with slightly updated data model and specification of notification system.  |
| `1.0`   | 2026-01-25 | Final Version.                                       |
| `1.0.1` | 2026-01-30 | Editorial update (licensing and reuse clarification) |


## 1. Introduction and Overview

The present document specifies selected technical and procedural aspects related to the **notification and
subsequent publication of Provider information** according to [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) and the Commission Implementation Regulation
(CIR) [(EU) 2024/2980](https://eur-lex.europa.eu/eli/reg_impl/2024/2980/oj) of 28 November 2024 laying down rules for the application of Regulation
[(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) of the European Parliament and of the Council as regards **notifications to the Commission
concerning the European Digital Identity Wallet (EUDIW) ecosystem**.

This document also covers the **notification of Public Sector Electronic Attestation of Attributes (PubEAA) Providers**
according to Article 3 (46) and Article 45f (3) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) together with the
 notification procedure according to Article 5 and Annex III of CIR [(EU) 2025/1569]( http://data.europa.eu/eli/reg_impl/2025/1569/oj).

To this extent the present document addresses the following types of Providers and notifications:

1. **Registrars and Registers of Wallet-Relying Parties** (see Article 5a (18) (a) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) and
 Annex II 1. of CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj)),
2. **Wallet Providers** and mechanism to validate authenticity and validity of Wallet Units (see Article 5a (18) (b)
 of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) and Annex
 II 2. of CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj)),
3. **Providers of Person Identification Data (PID)** and mechanisms enabling the authentication and validation of PID
 (see Article 5a (18) (c) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018)
 and Annex II 3. of CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj)),
4. **Providers of Wallet-Relying Party Access Certificates** (see Annex II 4. of CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj)),
5. **Public Sector Electronic Attestation of Attributes (PubEAA) Providers** (see Article 3 (46) and Article 45f (3) of
   [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018)) and
   Annex III of CIR [(EU) 2025/1569](http://data.europa.eu/eli/reg_impl/2025/1569/oj)

The following notification procedures are out of scope of the present document, as they are well
established already:

* Notification of **eID Schemes** according to Article 9 of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018)
* Notification of **Conformity Assessment Bodies** according to Article 20 (b) 1b of the amended Regulation [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018)
* Notification of **Supervisory Bodies for the European Digital Identity Wallet Framework** according to Article 46a of ([(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018)
* Notification of **Supervisory Bodies for Trust Services according** to Article 46b of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018)
* Notification of **Single Points of Contact** according to Article 46c of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018)
* Notification of **Qualified Trust Service Providers** according to Article 21 of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) to the national Supervisory
 Body and the related management of **Trusted lists** according to Article 22 of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018).

## 2. Data Model

The present data model has been created based on the implementing acts CIR
[(EU) 2024/2980](https://eur-lex.europa.eu/eli/reg_impl/2024/2980/oj),
CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj) and
CIR [(EU) 2025/1569]( http://data.europa.eu/eli/reg_impl/2025/1569/oj) while keeping
the [SEMIC Style Guide](https://semiceu.github.io/style-guide/1.0.0/index.html)
in mind and aligning it with the [Core Business Vocabulary](https://semiceu.github.io/Core-Business-Vocabulary/releases/2.2.0/) as much as reasonable.

![ProviderDataModel](img/ts2-eudi-providers.svg)

As outlined in the above figure, it contains the following **main classes**:

* **LegalEntity**
* **Provider**
* **WRPRegistrar**
* **WalletProvider**
* **PIDProvider**
* **PubEEAProvider**
* **TrustServiceProvider** with its subclass **WRPAccCertProvider**
* **WalletRelyingParty**

These classes are defined using the following **auxiliary classes**

* **Document**
* **Identifier**
* **Law**
* **LegalPerson**
* **NaturalPerson**
* **Policy**
* **TrustService**
* **WalletSolution**

### 2.1 LegalEntity

The ``LegalEntity`` class provides the basis for the derivation of the ``Provider`` class, which is the central class of the present data model.
Its design takes into account that all the involved legal entities may either be legal persons or natural persons and this class assembles the
generic attributes related to a legal entity, which are relevant here. It contains the attributes specified in the following table:


| Attribute          | Multiplicity | Type | Description  |
|--------------------|--------------|------|--------------|
| `legalPerson`      | [0..1]       | [*LegalPerson*](#294-legalperson)      | specifies the specific attributes of a **legal person**. This attribute is present, if and only if the legal entity is a private or public legal person. |
| `naturalPerson`    | [0..1]       | [*NaturalPerson*](#295-naturalperson) | specifies the specific attributes of a **natural person**. This attribute is present, if and only if the legal entity is a natural person. |
| `identifier`     | [0..*]       | [*Identifier*](#292-identifier)        | may be present in order to specify one or more **identifiers** of the legal entity, as stated in an official record together with the identification data of that official record, if applicable. |
| `postalAddress` | [0..*]       | *string*                               | may be present in order to specify the **postal address** of the legal entity in line with clause 6.6.1 of recommendation [ITU-T X.520](https://www.itu.int/rec/T-REC-X.520/en). |
| `country`        | [1..1]       | *string*                               | specifies the alpha-2 **country code** according to [ISO 3166-1](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/39) of the country in which the legal entity is established, or the string "EU" for providers operating on a European level. |
| `email`          | [0..*]       | *string*                               | may be present in order to specify one or more **email addresses** according to [RFC 5322](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/401) of the legal entity. |
| `phone`          | [0..*]       | *string*                               | may be present in order to specify one or more **phone numbers** of the legal entity starting with the ‘+’ symbol as the international code prefix and the country code, followed by numbers only as specified in [RFC 2806](https://www.rfc-editor.org/rfc/rfc2806.html).|
| `infoURI`        | [0..*]       | *string*                               | may be present in order to specify one or more Unique Resource Identifiers (URIs) according to [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.html) with **webpages for information** about the legal entity. |

### 2.2 Provider

The ``Provider`` class inherits all attributes from the more general ``LegalEntity`` class specified above and is the central class of the
data model from which the other classes for the individual provider classes derive. It contains the attributes specified in the following table:

| Attribute           | Multiplicity | Type                    | Description  |
|---------------------|--------------|-------------------------|--------------|
| `providerType` | [1..1]      | *string*                | specifies the **type** of the provider according to its sub-class, as it is necessary for the specification of the API in [clause 3.2](#3-2-components-and-interfaces), whereas the following values are defined in the present document: <ul><li>`WRPRegistrar`</li><li>`WalletProvider`</li><li>`PIDProvider`</li><li>`PubEEAProvider`</li>`WRPAccCertProvider`</li><li>`WalletRelyingParty`</li></ul>  |
| `policy`          | [1..*]       | [*Policy*](#295-policy) | specifies the type of the policy and the Uniform Resource Locator (URL) according to [RFC 1738]((https://www.rfc-editor.org/rfc/rfc1738.html)) of a webpage where the relevant **service policy** (e.g. registration policy or trust service policy) or **privacy policy** or **terms and conditions statement** of the provider are located, where applicable. |
| `x5c`             | [0..*]       | *string*                | specifies a sequence of **X.509 certificate chains** according to [RFC 7515](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/311), where each certificate chain is compliant to [RFC 3647](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/78) and [RFC 5280](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/77), if used by the provider for all its services. Specifying more than one certificate chain here allows to support key rollover procedures. In case of a ``TrustServiceProvider`` this element is not present, as the relevant certificate chains are assigned to the individual ``TrustService``. |

### 2.3 WRPRegistrar

According to Article 5a (18) (a) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018), the Member States are required to notify the European Commission
about the **Registrars for Wallet-Relying Parties**, which are responsible for establishing and maintaining the lists of
relying parties (**Registries of Wallet-Relying Parties**), that rely on European Digital Identity Wallets in
accordance with Article 5b (5) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) and the location of that lists.

According to Annex II 1. of CIR [(EU) 2024/2980](https://eur-lex.europa.eu/eli/reg_impl/2024/2980/oj), the EU Member States shall provide for each register and registrar the following information to
the European Commission:

* (a) the name of the register;
* (b) at least one URL where the register is available for access, which shall use state of the art transport layer encryption;
* (c) the name of the registrar that is responsible for that register;
* (d) where applicable, the registration number of the registrar;
* (e) the Member State in which the registrar is established;
* (f) the contact email and contact phone number of the registrar, for matters related to the register;
* (g) where applicable, the URL of a webpage for additional information about the registrar and the register;
* (h) the URL of the webpage where the registration policy that applies to the register and related information are located;
* (i) one or more certificates compliant with [RFC 3647](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/78) which can be used to verify the signature or seal created by the registrar on the register data and for which the certified identity data include the name of the registrar, and where applicable, the registration number of the registrar, as provided in points (c) and (d), respectively.

The ``WRPRegistrar`` class is used for this kind of notification and inherits all attributes from the ``Provider``
class. In addition to the attributes of the ``Provider`` class it contains the attributes specified in the following table:

| Attribute | Multiplicity | Type | Description |
|---------------|--------------|----------|-------------|
| `register` | [1..1] | *string* | specifies the **name of the register**. |
| `registerURI` | [1..*] | *string* | specifies the **URL where the register is available for access**. |

### 2.4 WalletProvider

According to Article 5a (18) (b) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018), the Member States are required to notify to the European Commission
about the **Wallet Providers**, which are the bodies responsible for the provision of European Digital Identity Wallets in accordance with Article 5a (1).
In line with Article 5a (18) (e) the notified information shall also include information about the mechanism for the validation of the authenticity
and validity of European Digital Identity Wallets.

According to Annex II 2. of CIR [(EU) 2024/2980](https://eur-lex.europa.eu/eli/reg_impl/2024/2980/oj), the EU Member States shall provide for each wallet provider the following information to
the European Commission:

* (a) the name of the wallet provider;
* (b) where applicable, the registration number of the wallet provider;
* (c) where applicable, the name of the body responsible for the provision of the wallet solution;
* (d) the Member State in which the wallet provider is established;
* (e) the contact email and contact phone number of the wallet provider, for matters related to the wallet solutions
 it provides;
* (f) where applicable, the URL of the webpage for additional information about the wallet provider and the wallet solution;
* (g) the URL of the webpage where the policies, terms and conditions of the wallet provider that apply to the provision
 and use of the wallet solution it provides are located;
* (h) one or more certificates compliant with [RFC 3647](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/78) that can be used to authenticate and validate the
 components of the wallet unit the wallet provides, and for which the certified identity data includes the name,
 and where applicable, the registration number of the wallet provider, as specified in points (a) and (b), respectively;
* (i) for each wallet solution provided by the wallet provider, the name and the reference number of the wallet solution
 it provides, as the Commission shall publish this information in the Official Journal of the European Union pursuant
 to Article 5d of Regulation [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018).

The ``WalletProvider`` class is used for this kind of notification and inherits all attributes from the ``Provider``
class. In addition to the attributes of the ``Provider`` class it contains the attributes specified in the following table:

| Attribute       | Multiplicity | Type  | Description  |
|-----------------|--------------|-------|--------------|
| `walletSol` | [1..*]      | [*WalletSolution*](#297-walletsolution) | specifies details with respect to the **wallet solution**. |

### 2.5 PIDProvider

According to Article 5a (18) (c) of Regulation [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018), the Member States are required to notify to the European Commission the
**Provider of Person Identification Data (PID)**, which are responsible for ensuring that the PID is associated with
the European Digital Identity Wallet in accordance with Article 5a (5) (f). In line with Article 5a (18) (d) the
notified information shall contain information, which allows to validate the PID.

According to Annex II 3. of CIR [(EU) 2024/2980](https://eur-lex.europa.eu/eli/reg_impl/2024/2980/oj), the EU Member States shall provide for each PID-Provider the
following information to the European Commission:

* (a) the name of the provider of person identification data;
* (b) where applicable, a registration number of the provider of person identification data;
* (c) where applicable, the name of the body responsible for ensuring that the person identification data is associated with the wallet unit;
* (d) the Member State in which the provider of person identification data is established;
* (e) the contact email and contact phone number of the provider of person identification data, for matters related to the person identification data it provides;
* (f) where applicable, the URL of the webpage that contains additional information about the person identification data provider;
* (g) the URL of the webpage that contains the policies, terms and conditions of the provider of person identification data that apply to the provision and use of the person identification data it provides;
* (h) one or more certificates compliant with [RFC 3647](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/78l) that can be used to verify the signature or seal created by the provider of person identification data on the person identification data it provides, and for which the certified identity data include the name, and where applicable, the registration number of the person identification data provider, as specified in points (a) and (b), respectively.

The ``PIDProvider`` class is used for this kind of notification and inherits all attributes from the ``Provider``
class. In addition to the attributes of the ``Provider`` class it contains the attributes specified in the following table:

| Attribute | Multiplicity | Type | Description |
|-----------|--------------|------|-------------|
| `PIDIssuer` | [0..1] | *string* | specifies the **name of the body responsible** for ensuring that the person identification data is associated with the wallet unit. |

### 2.6 PubEEAProvider

According to Article 45f (3) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) the
Member States shall notify **Public Sector Electronic Attestation of Attribute (PubEAA) Providers**
according to Article 3 (46) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) to the Commission. That notification shall include a conformity assessment report issued by a
conformity assessment body confirming that the requirements set out in Article 45f (1), (2) and (6) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) are met. The Commission shall
make available to the public, through a secure channel, the list of public sector bodies referred to in Article 3 (46) [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) in
electronically signed or sealed form suitable for automated processing. Further details are
specified in Article 5 and Annex III of CIR [(EU) 2025/1569]( http://data.europa.eu/eli/reg_impl/2025/1569/oj).

The ``PubEEAProvider`` class is used for this kind of notification and inherits all attributes from the ``Provider``
class, which in turn inherits all attributes from the ``LegalEntity`` class. Note, that a ``PubEEAProvider`` is always a public sector legal
entity, which is established by some public law and hence the element ``establishedByLaw`` in ``LegalPerson`` shall always be present.  
In addition to the attributes of the ``Provider`` class the ``PubEEAProvider`` class contains the attributes specified in the following table:

| Attribute | Multiplicity | Type | Description |
|-----------|--------------|------|-------------|
| `EEAIssuer` | [0..1] | [*Provider*](#22-provider)  | specifies the details of the **EEA issuer**, if it is not identical to the public sector body, which is responsible for the authentic source indicated in the ``Provider`` class. |
| `car` | [1..*] | [*Document*](#291-document) | contains the **conformity assessment report** mentioned in Article 45f (3) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) consisting of one or more ``Document`` elements. |

### 2.7 WRPAccCertProvider

The ``WRPAccCertProvider`` class is used for the notification of issuers of access certificates
for wallet-relying parties as required by Annex II 4. of CIR [(EU) 2024/2980](https://eur-lex.europa.eu/eli/reg_impl/2024/2980/oj).

It simply inherits the attributes from the ``TrustServiceProvider`` class, which in turn inherits
the attributes from the ``Provider`` class and ``LegalEntity`` class and adds one or more `trustSrv`
attributes, as specified in the following table:

| Attribute | Multiplicity | Type    | Description |
|-----------|--------------|---------|-------------|
| `trustSrv` | [1..*] | [*TrustService*](#296-trustservice) | specifies the **details of the trust service**. |

According to Article 5a (18) (d) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018)
the EU Member States shall notify the European Commission about the mechanism for the validation of the
identity of the Wallet-Relying Parties. As stipulated by the implementing acts
according to Article 5a (23) [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018), i.e.
the CIRs [(EU) 2024/2977](http://data.europa.eu/eli/reg_impl/2024/2977/oj), [(EU) 2024/2979](http://data.europa.eu/eli/reg_impl/2024/2979/oj), [(EU) 2024/2980](http://data.europa.eu/eli/reg_impl/2024/2980/oj)
and [(EU) 2024/2982](http://data.europa.eu/eli/reg_impl/2024/2982/oj), the validation of the identity of the Wallet-Relying Parties is envisioned to take place using
so called "Wallet-Relying Party Access Certificates", which are defined to be certificates for electronic seals or signatures
authenticating and validating the wallet-relying party issued by specific providers, which are mandated by the EU Member
State.

According to Annex II 4. of [(EU) 2024/2980](http://data.europa.eu/eli/reg_impl/2024/2980/oj), the
EU Member States shall notify the following information for each Provider of Wallet-Relying Party
Access Certificates to the European Commission:

* (a) the name of the provider of wallet-relying party access certificates;
* (b) where applicable, a registration number of the provider of wallet-relying party access certificates;
* (c) the Member State in which the provider of wallet-relying party access certificates is established;
* (d) the contact email and contact phone number of the provider of wallet-relying party access certificates,
 for matters related to the access certificates it provides to wallet-relying parties;
* (e) where applicable, the URL of the webpage of the provider of wallet-relying party access certificates that
 contains additional information about the provider and the access certificates it provides to wallet-relying parties;
* (f) the URL of the webpage that contains the policies, terms and conditions that apply to the provision and use of
 the access certificates it provides to wallet-relying parties;
* (g) one or more certificates compliant with [RFC 3647](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/78) that can be used to verify the signature or seal created
 by the provider of wallet-relying party access certificates on the access certificate it provides to wallet-relying
 parties, with, where applicable, the information required to distinguish wallet-relying party access certificates
 from other certificates.

### 2.8 Auxiliary Classes

#### 2.8.1 Document

The ``Document`` class is used within the definition of the [``PubEAAProvider``](#26-pubeeaprovider) class. It contains the attributes specified in the following table:

| Attribute | Multiplicity | Type | Description |
|-----------|--------------|------|-------------|
| `type` | [1..1] | *string* | is the **MIME type** of the document according to [RFC 2046](https://www.rfc-editor.org/rfc/rfc2046.html). |
| `content` | [1..1] | *string* | is the **content of the document** encoded as specified by its MIME type. |

#### 2.8.2 Identifier

The ``Identifier`` class is used within the definition of the [``LegalEntity``](#21-legalentity) class.
It contains the attributes specified in the following table:

| Attribute | Multiplicity | Type | Description  |
|-----------|--------------|------|--------------|
| `type` | [1..1] | *string* | is the **type** of the identifier specified by a URI according to [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.html), whereas the following URIs are defined in the present document: <ul><li>http://data.europa.eu/eudi/id/EORI-No - Economic Operator Registration and Identification Number (**EORI-No**) according to [(EU) No 1352/2013](http://data.europa.eu/eli/reg_impl/2013/1352/oj)</li> <li>http://data.europa.eu/eudi/id/LEI - Legal Entity Identifier (**LEI**) according to [(EU) No 2022/1860](http://data.europa.eu/eli/reg_impl/2022/1860/oj) and [ISO 17442-1](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/483)</li><li>http://data.europa.eu/eudi/id/EUID - European Unique Identifier (**EUID**) according to [(EU) 2020/2244](http://data.europa.eu/eli/reg_impl/2020/2244/oj) and [(EU) 2021/1042](http://data.europa.eu/eli/reg_impl/2021/1042/oj)</li> <li>http://data.europa.eu/eudi/id/VATIN - Value Added Tax Identification Number (**VATIN**) according to the [Council Directive 2006/112/EC](http://data.europa.eu/eli/dir/2006/112/oj)</li> <li>http://data.europa.eu/eudi/id/TIN - [Taxpayer Identification Number (**TIN**)](https://taxation-customs.ec.europa.eu/online-services/online-services-and-databases-taxation/taxpayer-identification-number-tin_en)</li> <li><http://data.europa.eu/eudi/id/Excise> - **Excise Number** according to Art. 2 (12) of the Council Regulation [(EC) No. 389/2012](http://data.europa.eu/eli/reg/2012/389/oj) </li> </ul>. |
| `identifier` | [1..1] | *string* | is the **identifier**, which identifies the ``LegalEntity`` under consideration. |

#### 2.8.3 Law

The ``Law`` class is used within the definition of the [``LegalPerson``](#284-legalperson)
class. It contains the attributes specified in the following table:

| Attribute | Multiplicity | Type | Description |
|-----------|--------------|------|-------------|
| `legislativeIdentifier` | [1..1]       | *string* | specifies the **legislative act** in form of a URI according to [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.html), such as the [European Legislation Identifier (ELI)](https://eur-lex.europa.eu/content/help/eurlex-content/eli.html?locale=en) for example. |
| `legalBasis` | [0..1]      | *string* | may be present to specify the **legal basis** more precisely, if applicable. |

#### 2.8.4 LegalPerson

The ``LegalPerson`` class is used within the definition of the [``LegalEntity``](#21-legalentity)
class and allows to specify the attributes of a legal person. It contains the attributes specified in the following table:

| Attribute          | Multiplicity  | Type  | Description |
|--------------------|---------------|-------|-------------|
| `legalName`        | [1..*]        | *string*           | specifies the **legal name** of the legal person, as specified in an official record. |
| `establishedBylaw` | [0..*]        | [*Law*](#283-law) | may be present in case of a **public sector body responsible for an authentic source** according to Article 3 (46) and (47) of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018) to specify the **legal basis** upon which it is established as authentic source. This information should in particular be present, if there is no other unambiguous identifier for the public sector body and authentic source under consideration. |

#### 2.8.5 NaturalPerson

The ``NaturalPerson`` class is used within the definition of the [``LegalEntity``](#21-legalentity)
class and allows to specify the attributes of a natural person. It contains the attributes specified in the following table:

| Attribute          | Multiplicity | Type                | Description  |
|--------------------|--------------|---------------------|--------------|
| `givenName`        | [1..1]       | *string*            | specifies the current **first name(s)** of the natural person including middle name(s) where applicable, as specified in official records and the set of person identification data according to the Annex of [(EU) 2024/2977]( http://data.europa.eu/eli/reg_impl/2024/2977/oj). |
| `familyName`       | [1..1]       | *string*            | specifies the current **last name(s)** or surnames of the natural person, as specified in official records and the set of person identification data according to the Annex of [(EU) 2024/2977]( http://data.europa.eu/eli/reg_impl/2024/2977/oj).|
| `dateOfBirth`      | [0..1]      | *string*            | specifies the **date of birth** of the natural person, as specified in official records and the set of person identification data according to the Annex of [(EU) 2024/2977]( http://data.europa.eu/eli/reg_impl/2024/2977/oj), if present. |
| `placeOfBirth` | [0..1]      | *string*            | specifies the **place of birth** of the natural person, as specified in official records and the set of person identification data according to the Annex of [(EU) 2024/2977]( http://data.europa.eu/eli/reg_impl/2024/2977/oj), if present. |

#### 2.8.6 Policy

The ``Policy`` class is used within the definition of the [``Provider``](#21-provider) class. It contains the attributes specified in the following table:

| Attribute | Multiplicity | Type | Description |
|-----------|--------------|------|-------------|
| `type` | [1..1] | *string* | specifies the **type of the policy** in form of a URI according to [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.html), whereas the following URIs are defined in the present document: <ul><li>http://data.europa.eu/eudi/policy/trust-service-practice-statement - is a **Trust Service Practice statement** according to clause 6.1 of [ETSI EN 319 401](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/140).</li><li>http://data.europa.eu/eudi/policy/terms-and-conditions - is a **Terms and Conditions** statement according to clause 6.2 of [ETSI EN 319 401](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/140).</li><li>http://data.europa.eu/eudi/policy/privacy-statement - is a **Privacy Statement** to fulfil the information requirements of Article 12 ff [(EU) 2016/679](http://data.europa.eu/eli/reg/2016/679/oj).</li><li>http://data.europa.eu/eudi/policy/privacy-policy - is a **Privacy Policy** according to the clauses 3.14 and 5.6 of [ISO/IEC 29100](https://www.iso.org/standard/85938.html).</li><li>http://data.europa.eu/eudi/policy/registration-policy - is a **Registration Policy** according to Article 4 of CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj).</li> </ul> |
| `policyURI` | [1..1] | *string* | specifies the **policy URI** in form of a URL according to [RFC 1738](https://www.rfc-editor.org/rfc/rfc1738.html) where the policy is published. |

#### 2.8.7 TrustService

The **TrustService** class is used within the definition of the [TrustServiceProvider](#286-trustservice) class. It contains the attributes specified in the following table:

| Attribute | Multiplicity | Type | Description |
|-----------|--------------|------|-------------|
| `type` | [1..1]       | *string* | specifies the **Service type identifier** of the Trust Service in Form of a URI according to [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.html), whereas the set of admissible URIs is defined in clause 5.5.1 of [ETSI TS 119 612](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/41). |
| `srvName` | [1..1]       | *string* | specifies the **TSP trade name** of the Trust Service according to clause 5.4.2 of [ETSI TS 119 612](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/41). |
| `x5c` | [1..*]      | *string* | specifies the set of **Service digital identities** of the Trust Service according to clause 5.5.3 of [ETSI TS 119 612](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/41) in form of X.509 certificate chains encoded in Base 64. |
| `status` | [1..1]       | *string* | specifies the **Service current status** of the Trust Service according to clause 5.5.4 point i) of [ETSI TS 119 612](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/41). |
| `statusDate` | [1..1]       | *string* | specifies the **Current status starting date and time** of the Trust Service according to clause 5.5.5 of [ETSI TS 119 612](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/41) using the format defined in [ISO 8601-1](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/405). |
| `srvURI` | [1..*]       | *string* | specifies the **Service supply points** of the Trust Service according to clause 5.5.7 of [ETSI TS 119 612](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/41). |
| `srvQualifier` | [0..*]       | *string* | specifies the **Service information extensions** of the Trust Service according to clause 5.5.9 of [ETSI TS 119 612](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/41). |


#### 2.8.8 WalletSolution

The **WalletSolution** class is used within the definition of the [WalletProvider](#24-walletprovider) class. It contains the attributes specified in the following table:

| Attribute | Multiplicity | Type | Description  |
|-----------|--------------|------|--------------|
| `solProvider` | [0..1] | *string* | specifies the **name of the body responsible** for the provision of the wallet solution, if applicable. |
| `walletName` | [1..1] | *string* | specifies the **name of the wallet solution**. |
| `refNum` | [1..1] | *string* | specifies the **reference number** of the wallet solution, which is published in the Official Journal of the European Union according to Article 5d of [(EU) No 910/2014](https://eur-lex.europa.eu/legal-content/DE/TXT/?uri=CELEX%3A02014R0910-20241018). |

## 3. Notification System

As defined in Article 3 (1) of CIR [(EU) 2024/2980](http://data.europa.eu/eli/reg_impl/2024/2980/oj), the Commission
will make available to Member States a *Secure Electronic Notification System* enabling Member States to notify
the information on the bodies and mechanisms referred to in Article 5a (18) of [(EU) No 910/2014](https://www.eid.as/#article5a)
and specified in more detail in the Annexes of CIR [(EU) 2024/2980](http://data.europa.eu/eli/reg_impl/2024/2980/oj) and the present document.

The Member States will submit the relevant information according to Article 4 of CIR [(EU) 2024/2980](http://data.europa.eu/eli/reg_impl/2024/2980/oj)
and Article 5 of CIR [(EU) 2025/1569]( http://data.europa.eu/eli/reg_impl/2025/1569/oj) to the Commission and the
Commission will in turn publish the relevant information according to Article 4 of [(EU) 2024/2980](http://data.europa.eu/eli/reg_impl/2024/2980/oj)
in form of suitable lists according to Article 5 of CIR [(EU) 2024/2980](http://data.europa.eu/eli/reg_impl/2024/2980/oj)
and [ETSI TS 119 602](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/278).

### 3.1 System Architecture

The envisioned system architecture for the Secure Electronic Notification System described in the present document is fairly simple and mainly consists of

* the *Secure Electronic Notification System* operated by the European Commission and
* decentral *Client Components* operated  by the Member States and other stakeholders within the EUDI ecosystem, which rely on the lists of trusted entities respectively,

as outlined in the following figure.

![NotificationSystem](img/ts2-eudi-notification-system.svg)

### 3.2 Interfaces

The Secure Electronic Notification System has the following interfaces:

1. the *Application Programming Interface (API)* as specified in [*clause 3.3*](#33-application-programming-interface) below and within the [OpenAPI](https://spec.openapis.org/oas/latest.html)-specification provided in [Annex A.2](#a2-openapi-specifications-normative)
2. a suitable *Web Interface* which is at least accessible by the authorized member state representatives and
3. a web endpoint,  where the lists of trusted entities according to Article 5 of CIR [(EU) 2024/2980](http://data.europa.eu/eli/reg_impl/2024/2980/oj) and [ETSI TS 119 602](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/278) and related information is available.

### 3.3 Application Programming Interface

#### 3.3.1 Methods

The [OpenAPI](https://spec.openapis.org/oas/latest.html)-specification for the methods outlined in the following is provided in [Annex A.2](#a2-openapi-specifications-normative).

##### 3.3.1.1 `/v1/providers` (GET)

This method allows obtain a list of existing providers according to specified filter criteria. If none filter critaria are given, all providers will be returned.

##### 3.3.1.2 `/v1/{country}/providers` (POST)

This method allows to create a provider.

##### 3.3.1.3 `/v1/{country}/providers/{providerId}` (GET)

This method allows obtain the full data set of an existing provider.

##### 3.3.1.4 `/v1/{country}/providers/{providerId}` (PUT)

This method allows update the data of an existing provider.
An `ETag` must be retrieved first, such that parallel updates can be
prevented by the server. The server MAY limit which modifications can be made.

##### 3.3.1.5 `/v1/{country}/providers/{providerId}` (DELETE)

This method allows to delete an existing provider.

#### 3.3.2 Authentication and authorisation

The write methods (POST, PUT, DELETE) SHALL be only accessible for authenticated and authorised

API clients. The details with respect to authentication and authorisation are beyond the
scope of the present specification.

#### 3.3.3 Protection against DDOS attacks and accidental overuse of the GET methods

The API implementation SHALL take into consideration the vulnerabilities that an open,
comprehensive query interface enables for both malicious or non-malicious users of the interface.
This for example includes the possibility of Distributed Denial-of-Service (DDoS) attacks.

##### 3.3.3.1 DDoS attack protection guidelines

Following policy/recommendation SHOULD be implemented (with solution-fitted combinations) in
the API implementation to protect the API from DDoS attacks:

- Use of **a Cloud-based DDoS Protection Service/CDN** as the best first line of defense for volumetric attacks.
- Use of an **API Gateway or Load Balancer with Rate Limiting** is crucial for fight against application-layer attacks targeting the API.
- Deploy a **Web Application Firewall (WAF)** to protect against other application-layer threats and to complement rate limiting.
- Ensure your infrastructure is scalable by using e.g. **Load Balancers** to handle legitimate bursts and some attack traffic.
- Utilise **Caching** heavily for the GET endpoints to reduce backend load - many responses might be static or change infrequently, and caching the responses at the CDN, load balancer or application level means the API logic and database will receive less requests.
- Have robust **API monitoring** for request rates, latency, error rates, server resource utilisation and source IP addresses with proper alerting in place.
- Utilise **network segmentation** - e.g., isolate the API servers in a private subnet behind the WAF, and from the actual provider database to improve robustness of the network architecture against hostile attacks.

##### 3.3.3.2 Protection against unintentional misuse

The implementers SHOULD consider how to limit the load of the API from legitimate but unintentionally
complex or frequent API calls to the query endpoints. Possible solutions for this are:

- The GET endpoints SHOULD be provided with e.g. rate limiter for calls coming in from an individual IP address.
- The GET `/v1/providers` endpoint is an example of an API method that allows throwing a lot of computational
  load for a server serving data from a potentially large database of providers information via requesting
  unnecessarily complex query parameter combinations. Best practices for limiting the number of query parameters
  in a valid API call SHOULD be tested with an outcome as a documented guideline for proper use
  of the API. Improperly configured API calls can be rejected at selected level of the backend
  architecture.

The implementors SHOULD publish in their respective API documentation what limits it has configured
regarding rate limits for legitimate use, or use of the query parameters.

## Annex A

### A.1 JSON-Schema (normative)

The file [``ts-2-eudi-provider.json``](api/ts2-eudi-provider.json) contains the JSON-Schema definitions corresponding to the Data Model specified in [clause 2](#2-data-model) above.

### A.2 OpenAPI-Specifications (normative)

The file [``ts2-openapi-eudi-provider.json``](api/ts2-openapi-eudi-provider.json) contains the JSON-based [OpenAPI](https://spec.openapis.org/oas/latest.html)-specification of the API specified in [clause 3](#3-notification-system) above.

### A.3 XML-Schema (informative)

The file [``ts2-eudi-provider.xsd``](api/ts2-eudi-provider.xsd) contains the XML-Schema definitions corresponding to the Data Model specified in [clause 2](#2-data-model) above.
