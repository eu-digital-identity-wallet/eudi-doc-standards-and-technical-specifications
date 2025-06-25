<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification of Common Interface for reporting of Relying Parties to Data Protection Authorities

## Abstract

The present document specifies the common protocols and interfaces according to Article 5a (5) (a) (x) of
[(EU) No 910/2014](http://data.europa.eu/eli/reg/2014/910/2024-10-18) used by a Wallet Unit for reporting a Wallet-Relying Party 
to the competent national Data Protection Authority where an allegedly unlawful or 
suspicious request for data is received.

### [GitHub Discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/discussions/categories/technical-specification-discussion)

## Versioning

| Version | Date        | Description |
|---|---|---|
| `0.1`   | 27.05.2025 | Initial version for first discussions. |
| `0.2`  | 12.06.2025 | Update after first discussions. |
| `0.9`  | 22.06.2025 | Update after second focus meeting. |

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
"Key words for use in RFCs to Indicate Requirement Levels" [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119).
The interpretation should only be applied when the terms appear in
all capital letters.

## 1. Introduction

The present document specifies the **common protocols and interfaces** according to Article 5a (5) (a) (x) of
[(EU) No 910/2014](http://data.europa.eu/eli/reg/2014/910/2024-10-18) used by a Wallet Unit for invoking a suitable external application,
such as a browser or an email client, to **report a Wallet-Relying Party** (WRP) to the competent 
national Data Protection Authority (DPA) where an allegedly unlawful or suspicious request for data is received.

The general approach with respect to the protocols and interfaces specified in the present document are 
based on the result of the related discussions documented in the corresponding [L+M-Discussion-Paper](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/discussion-topics/l%2Bm-data-deletion-and-reporting-of-wrp-to-dpa.md). 

## 2. Requirements

### 2.1 Legal Requirements

Article 5a (5) of the regulation [(EU) No 910/2014](http://data.europa.eu/eli/reg/2014/910/2024-10-18) requires that 

>European Digital Identity Wallets shall, in particular:
>
>(a) support common protocols and interfaces:
>
> [...]
>
> (x) for reporting a relying party to the competent national data protection authority where an allegedly unlawful or suspicious request for data is received.

Article 7 of [(EU) 2024/2982](http://data.europa.eu/eli/reg_impl/2024/2982/oj) stipulates the following:

>1. Wallet providers shall ensure that wallet units allow wallet users to easily report wallet-relying parties to supervisory authorities established under Article 51 of Regulation (EU) 2016/679.
>
>2. Wallet providers shall implement the protocols and interfaces for reporting wallet-relying parties in compliance with national procedural laws of the Member States.
>
>3. Wallet providers shall ensure that wallet units allow wallet users to substantiate the reports, including by attaching relevant information to identify the wallet-relying parties, and the wallet users’ claims in machine-readable format.


### 2.2 High-Level Requirements

The set of high-level requirements after the discussions documented in the [L+M-Discussion-Paper](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/discussion-topics/l%2Bm-data-deletion-and-reporting-of-wrp-to-dpa.md) and
the two focus meeting for the present specification, which added `RPT_DPA_07` and `RPT_DPA_08` are as follows:


| **Index**  | **Requirement specification** |
|---|---|
| RPT_DPA_01 | When prompted by the User, a Wallet Unit SHALL provide the contact details of the DPA, which supervises the Relying Party, if available, and SHALL make it easy for the User to send a report of allegedly unlawful or suspicious Relying Party presentation requests to this DPA. If these are not available, the Wallet Unit SHALL provide the contact details of the DPA of the region in which the Wallet Provider is residing. In addition, the Wallet Unit MAY also provide contact details of other DPAs taken from the "European Data Protection Board" website (https://www.edpb.europa.eu/about-edpb/about-edpb/members_en), and allow the User to choose a DPA to continue the reporting process. |
| RPT_DPA_02 | The User interface enabling a User to start the process of reporting a Wallet-relying Party to a DPA SHALL be accessible via the log provided by the Wallet Unit.|
| RPT_DPA_04 | Wallet providers SHALL ensure that wallet units allow wallet users to substantiate the reports, including by attaching relevant information to identify the wallet-relying parties, and the wallet users’ claims in machine-readable format.</br> Note: The log kept by the Wallet Unit will be standardized and is machine-readable in order to enable data portability. An excerpt from this log therefore can be used to substantiate the report. |
| RPT_DPA_05 | A Wallet Unit SHALL log the initiation of a report sent to the DPA in a log file so that it can be presented to the User in the dashboard (as specified in [Topic 19](#a2319-topic-19---user-navigation-requirements-dashboard-logs-for-transparency)).  |
| RPT_DPA_06 | The Wallet Unit SHALL take the contact details of the DPA, which supervises the Relying Party, either (in this order) from a) included in  the RPRC in the log entry, b) included in the RPAC in the log entry, c) looked up by the Wallet Unit from the RP Registry, based on the Subject of the RPAC in the log entry. </br> The contact information includes  at least one of email address, phone number, or a URL of a webform. |
| RPT_DPA_07 | The text in the `subject` of the mail prepared by the Wallet Unit according to `RPT_DPA_06` SHALL clearly indicate that the purpose of the mail is to report a WRP to the DPA and state that an allegedly unlawful or suspicious request for data has been received. |
| RPT_DPA_08 | The text in the `subject` of the mail prepared by the Wallet Unit according to `RPT_DPA_06` SHOULD indicate the identity of the WRP and also an involved intermediary, if applicable, which sent the allegedly unlawful or suspicious request. |
| RPT_DPA_09 | The text in the `body` of the mail prepared by the Wallet Unit according to `RPT_DPA_06` SHALL contain the necessary information to identify the WRP and also an involved intermediary, if applicable, and allows that the request can be handled by the DPA in an efficient manner. |

> **Open Issue 1**: The final specification SHOULD specify recommended templates for the 
> `subject` and the `body` of the mail prepared by a Wallet Unit according to `RPT_DPA_06` until `RPT_DPA_09`.
> Before this can happen, there is the need to resolve subtle technical issues and current
> **misalignments** with respect to the not yet completely achieved harmonization of the encoding of WRP identities in [OID4VP (clause 5.9.3)](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html#section-5.9.3),
> [ISO/IEC 18013-5](https://www.iso.org/standard/69084.html), [HAIP (clause 5)](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-1_0-03.html#section-5),
> [ETSI TS 119 411-8 (V0.0.2)](https://portal.etsi.org/webapp/WorkProgram/Report_WorkItem.asp?WKI_ID=74431), [ETSI TS 119 475 (V0.0.6)](https://portal.etsi.org/webapp/WorkProgram/Report_WorkItem.asp?WKI_ID=69585) and last but not least
> [TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md).

> **Open Issue 2**: As the finally published version of the implementing act [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj)
> does **not** mention in its ANNEX I (Information regarding wallet-relying parties) that the contact information of the competent DPA shall
> be included in the registered information of the WRP, and in a similar manner in the Wallet-Relying Party Registration Certificate (WRPRC) or the 
> Wallet-Relying Party Registration Certificate (WRPRC), the only direct legal requirements are as listed in section 2.1 above.
> Therefore it seems to be the obligation of the related technical specifications to ensure that these legal requirements can be fulfilled 
> on a technical level, and consequently it seems to be appropriate to **ensure** in [TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md)
> that the appropriate contact information of the DPA is available in the registration information of the WRP. Therefore, `RPT_DPA_01` and `RPT_DPA_06` 
> SHALL be adapted accordingly as soon as the change in  [TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md), 
> which ensures that the contact information of the DPA is available in the WRP registry information, has been implemented.

> **Open Issue 3**: The transaction log mentioned in `RPT_DPA_04`, from which a part can be used to substantiate 
> a report to a DPA as required in Article 7 Nr. 3 of [(EU) 2024/2982](http://data.europa.eu/eli/reg_impl/2024/2982/oj) 
> and which will also stored in the transaction log according to `RPT_DPA_05`, will be specified in a forthcoming 
> Technical Specification based on the results of the related discussion in [Topic H - Transaction logs kept by the Wallet](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/discussion-topics/h-transaction-logs-kept-by-the-wallet.md). 
> As soon as this specification is finished, the present document will be updated to include specific references to this document.

## 3. Specification of Protocols and Interfaces

### 3.1 System Architecture

As agreed upon within the discussions documented in the [L+M-Discussion-Paper](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/discussion-topics/l%2Bm-data-deletion-and-reporting-of-wrp-to-dpa.md) 
the general approach is to utilise the already existing interfaces and processes of the 
competent national data protection authorities as much as possible and abstain from the creation of 
additional technical interfaces.

The resulting system architecture is depicted in the following figure:

![Overview](img/ts8-architecture.svg) 

### 3.2 Interfaces

As shown in the figure above, there are the following interfaces:

* **(I1)** - is a user-friendly user interface, which allows to access the functionality of the Wallet Unit and which in particular
  allows to report a WRP from which an allegedly unlawful or suspicious request for data has been 
  received to the competent national DPA, or to another DPA chosen by the user. This 
  functionality shall be immediately accessible when such a request has been received or later based on the 
  information stored in the Transaction Log. Wallet units shall allow wallet users to 
  substantiate the reports by attaching relevant information to identify the WRP and the 
  requested claim names in machine-readable format. The technical details of this format will 
  be derived from the normalised format of the transaction log, which will be specified based 
  on the related discussions within the [H-Discussion-Paper](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/discussion-topics/h-transaction-logs-kept-by-the-wallet.md).

* **(I2)** - is the API to the National Register according to Article 3 of
  [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj), which allows
  to retrieve the relevant information from the `supervisoryAuthority` element within 
  the registered information for a WRP and its intermediary, if applicable.

> **Open Issue 4**: As there are EU Member States, such as Germany for example, which 
> enjoy the presence of multiple DPAs and hence there may be two different DPAs involved (i.e. 
> one for the WRP and one for the intermediary). Therefore, it needs to be clarified in [TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md) 
> how to handle such a case. Note, that this case also appears, if the WRP and its intermediary
> are not located in the same EU Member State and consequently supervised by different DPAs.

* **(I3)** - is a platform-specific interface, which allows to invoke a browser on the platform to open a specific
  URL of the DPA in charge of supervising the WRP, which has submitted an allegedly unlawful 
  or suspicious request.

* **(I4)** - represents a logical interface provided by the web application of the competent 
  national DPA responsible for the supervision of the WRP at the given URL contained in the first 
`supervisoryAuthority.infoURI`  of the Wallet-Relying party according to [STS-TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md).

* **(I5)** - is a platform-specific interface, which allows to invoke a mail client on the 
  platform of the user with a `mailto:` link with the email address of the competent national
  DPA, which supervises the WRP under consideration, and the intermediary, if applicable. 
  The email address may be provided as part of the WRPRC, if available, or the provided WRPAC, 
  or within the first `supervisoryAuthority.email` element within the registration 
  information of the WRP according to [STS-TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md). Note, that the Wallet Unit shall 
  also support the option to send a report to the DPA responsible for supervising 
  the Wallet Provider or to look up other data protection authorities from the website of the 
  "European Data Protection Board"  at https://www.edpb.europa.eu/about-edpb/about-edpb/members_en.

* **(I6)** - represents a logical interface given by the mail system of the DPA, which is 
  assumed to exist, if the registration information with respect to the 
  `supervisoryAuthority` element of the WRP according to [STS-TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md) contains 
  at least one `email` element. 

> **Open Issue 5**: [STS-TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md) 
> SHOULD contain a recommendation that DPAs SHOULD provide a web form for submission of reports, 
> similar to the complaints submission form mentioned in recital 141 of [(EU) 2016/679](http://data.europa.eu/eli/reg/2016/679/oj), 
> as we do not assume that there is a mail client on the platform, but only a browser. 
> Furthermore, we should also RECOMMEND in [STS-TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md) 
> that the `supervisoryAuthority.infoURI` or at least the `supervisoryAuthority.email` of 
> the DPA is part of the WRPRC, if it exists, and the WRPAC, as this is known to be available 
> in any case.

* **(I7)** - is a platform-specific interface, which allows to invoke the phone application of the platform in 
  order to call the competent national DPA.

* **(I8)** - is the phone system of the DPA, which SHALL exist, if there is only a `phone` number
  registered within the `supervisoryAuthority` element of [STS-TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md). 

### 3.3 Process

The process for reporting a WRP to the competent national DPA where an allegedly unlawful or
suspicious request for data has been received is completely analogous to the process for
requesting the deletion of data at a WRP according to [TS7](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts7-common-interface-for-data-deletion-request.md), where the essential difference is
that in this case the contact information of the DPA in `supervisoryAuthority` is gathered
in the first step before the corresponding application (i.e. browser, mail client or phone application)
on the platform of the user is invoked. In addition to the `subject` and the `body` of the pre-filled 
mail, there may also be an attachment with the request from the WRP as specified in `RPT_DPA_04`.  