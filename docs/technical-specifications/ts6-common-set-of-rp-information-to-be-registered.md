<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Common Set of Relying Party Information to be Registered

## Abstract

The present document specifies the common data set required for Relying Party Registration, as required by Regulation
[(EU) No. 910/2014](http://data.europa.eu/eli/reg/2014/910/oj), as augmented by the "European Digital Identity Regulation"
[(EU) 2024/1183](http://data.europa.eu/eli/reg/2024/1183/oj) and further detailed by the Commission Implementation Regulation
(CIR) [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj), as amended by CIR [(EU) 2026/1730](http://data.europa.eu/eli/reg_impl/2026/1730/oj).

### [GitHub discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/discussions/358)

## Licensing and Reuse

© European Union, 2025-2026.

This document is made available under the Creative Commons Attribution 4.0 International licence (CC BY 4.0), unless otherwise stated.

You may reuse this document provided that appropriate credit is given and any changes are indicated.

The full licence text is available at:
https://creativecommons.org/licenses/by/4.0/

## Versioning

| Version | Date        | Description |
|---------|-------------|------------------------|
| `0.1`   | 2025-04-25  | Initial version for first discussion  |
| `0.2`   | 2025-05-12  | Updated version from first commenting round |
| `0.3`   | 2025-05-23  | Updated version from second focus group meeting |
| `1.0`   | 2025-06-13  | Final version after the Coordination Group review period |
| `1.0.1` | 2026-01-30  | Editorial update (licensing and reuse clarification) |
| `1.1`   | 2026-06-10  | Updated version as outcome of topic refinement round |
| `1.2`   | 2026-08-04 | Update to align with changes introduced by CIR (EU) 2026/1730 |
| `1.2.1` | 2026-08-05 | Fix to new attribute naming and reverting removal of providedAttestation attribute |

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
"Key words for use in RFCs to Indicate Requirement Levels" [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).
The interpretation should only be applied when the terms appear in
all capital letters.

## 1. Introduction and Overview

The present document specifies the minimum common data set required from the wallet-relying parties in **Relying Party Registration**
according to the Regulation [(EU) No. 910/2014](http://data.europa.eu/eli/reg/2014/910/oj), as augmented by the "European Digital Identity Regulation"
[(EU) 2024/1183](http://data.europa.eu/eli/reg/2024/1183/oj), and further detailed by the Commission Implementation Regulation
(CIR) [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj), as amended by CIR [(EU) 2026/1730](http://data.europa.eu/eli/reg_impl/2026/1730/oj).

Relying Party Registration is introduced in Article 5b (1) of the Regulation [(EU) No. 910/2014](http://data.europa.eu/eli/reg/2014/910/oj), which
has been introduced with Regulation [(EU) 2024/1183](http://data.europa.eu/eli/reg/2024/1183/oj) and which reads as follows:

> (1) Where a relying party intends to rely upon European Digital Identity Wallets for the provision of public or private services by means of digital interaction, the relying party shall register in the Member State where it is established.

Article 5b (10) of the Regulation [(EU) No. 910/2014](http://data.europa.eu/eli/reg/2014/910/oj) explicitly mentions the case that the wallet-relying party may rely upon intermediaries:  

> (10) Intermediaries acting on behalf of relying parties shall be deemed to be relying parties and shall not store data about the content of the transaction

The details with respect to Relying Party Registration are further detailed in CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj) including the amendments from CIR [(EU) 2026/1730](http://data.europa.eu/eli/reg_impl/2026/1730/oj).

Article 3 Nr. 2 of CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj) states the following:

> the register shall include at least the information set out in Annex I.

However the legal text does not contain precise technical specifications and data models. Therefore the present document augments the legal text by pointing to the relevant technical specifications of the European Commission, which
in particular include [TS 5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md),
which inherits data structures from [TS 2](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts2-notification-publication-provider-information.md),
[TS 7](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts7-common-interface-for-data-deletion-request.md) and [TS 8](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts8-common-interface-for-reporting-of-wrp-to-dpa.md),
which impose requirements for additional data structures, which are already aligned with [ETSI TS 119 475](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/287).

## 2 Common Data Set

This section provides a table of minimum set of attributes, that SHALL be provided in Wallet-Relying Party
registration based on ruling of CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj) (Annex I), as augmented by
CIR [(EU) 2026/1730](http://data.europa.eu/eli/reg_impl/2026/1730/oj).

For guidance on the data format of each information attribute, the table rows contain

- the **Legal Basis** with a reference to Annex I of CIR [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj), as augmented by
  CIR [(EU) 2026/1730](http://data.europa.eu/eli/reg_impl/2026/1730/oj), if not explicitly specified otherwise, or
  other legal basis in which case there will be an explicit reference,
- a short name for the registration **Information**,
- a **Description** of the registration information,
- a reference to the **Attribute** within the class `WalletRelyingParty` in [TS5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md), which partly inherits data structures from [TS 2](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts2-notification-publication-provider-information.md) and
- additional **Notes**, if applicable.

| Legal Basis  | Information | Description  | Attribute in class </br> `WalletRelyingParty` in [TS 5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md) | Notes  |
|--------------|-------------|----|------------------|-----------|
| 1. | Name | The name of the wallet-relying party, as stated in an official record together with identification data of that official record, where applicable. If not applicable, the trade name according to paragraph 2 shall be used. | `legalName` for legal person, or </br> `givenName` and `familyName` for natural person | Please be reminded that according to the Article 3 (6) of [(EU) No. 910/2014](http://data.europa.eu/eli/reg/2014/910/oj) the wallet-relying party can be either a legal person or a natural person. |
| 2. | Trade Name | A user-friendly name of the wallet-relying party that can be either a trade name or service name that is recognisable to the user, if applicable. | `tradeName` |
| 3. | Identifier | Where applicable, one or more identifiers of the wallet-relying party, as stated in an official record together with identification data of that official record, expressed as: </br></br> (a) an economic operators registration and identification (‘EORI’) number as referred to in Commission Implementing Regulation [(EU) No 1352/2013](http://data.europa.eu/eli/reg_impl/2013/1352/oj);</br></br>(b) a registration number as registered in a national business register;</br></br>(c) a legal entity identifier (‘LEI’) as referred to in Commission Implementing Regulation [(EU) 2022/1860](http://data.europa.eu/eli/reg_impl/2022/1860/oj);</br></br>(d) a value-added tax (‘VAT’) registration number; </br>(e) an excise number as referred to in Article 2(12) of Council Regulation [(EU) No 389/2012](http://data.europa.eu/eli/reg/2012/389/oj);</br>(f) a tax reference number;</br></br>(g) an european unique identifier (‘EUID’) as referred to in Commission Implementing Regulation [(EU) 2021/1042](http://data.europa.eu/eli/reg_impl/2021/1042/oj);</br></br>(h) other national identifier or identifiers.| `identifier` | 1) If several identifiers are registered to the registry, at least one identifier SHALL be of type 'EUID' whenever available in the national (business) registry or registries.</br></br>2) If not available, such Member State SHALL specify and publish their national unique identifier scheme to be used for wallet-relying party identifiers.</br></br>3) See clause 9 of [(EU) 2021/1042](http://data.europa.eu/eli/reg_impl/2021/1042/oj) for the structure of the 'EUID' and be reminded, that the structure contains a 'Register Identifier', which allows to use different registers on the national level to create the 'EUID'.  </br></br> 4) Please refer to [TS 2](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts2-notification-publication-provider-information.md) (Clause 2.8.2) or [ETSI TS 119 475](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/287) (Table 2) for defined URIs which SHALL shall be used for registration of the `identifier` attribute. |
| 4. | Physical Address | The physical address where the wallet-relying party is established. | `postalAddress` |    |
| 5. | Info URI | Where applicable, a uniform resource locator (‘URL’) belonging to the wallet-relying party. | `infoURI` |
| 6. | *See paragraph 3.* | Where the `identifier` is expressed in accordance with points 3(a), (d), (f) or (h), the country indicator of the Member State where the wallet-relying party is established shall be prefixed using ISO 3166-1 Alpha 2 codes, with the exception of the country indicator for Greece which shall be ‘EL’. | *See paragraph 3.* |
| 7. | Contact Information | Contact information of the wallet-relying party, at least one of the following: </br></br> (a) a website where the wallet-relying party can be contacted for matters pertaining to provision of helpdesk and support; </br></br> (b) a phone number where the wallet-relying party can be contacted for matters pertaining to its registration and intended use of the wallet units; </br></br>(c) an email address where the wallet-relying party can be contacted for matters pertaining to its registration and intended use of the wallet unit. | (a) `supportURI` </br></br> (b) `phone` </br></br> (c) `email` | It is RECOMMENDED that wallet-relying parties register a `supportURI`. |
| 8. | Service Description | A description of the type of services the wallet-relying party provides. | `WalletRelyingPartyService.srvDescription` |
| 9. | Data Requested | For each intended use, a list of the data, including attestations and attributes, that the relying party intends to request, a user-friendly name and a technical name, the attestation type and any other syntaxes that the data is grouped under, in a machine-readable format for automated processing. | `WalletRelyingPartyService.IntendedUse.credentials` |
| 10. | Purpose  | For each intended use, a description of intended use of the data that the wallet-relying party intends to request from wallet units. | `WalletRelyingPartyService.IntendedUse.purpose` |
| 11. | Public Sector Body | An indication whether the wallet-relying party is a public sector body. | `isPSB` |
| 12. | Entitlements | The entitlement or entitlments of the wallet-relying party, that shall be expressed as follows: </br> (a) ‘Service_Provider’ to express the entitlement of the wallet-relying party as a provider of services; </br> (b) ‘QEAA_Provider’ to express the entitlement of the wallet-relying party as a qualified trust service provider issuing qualified electronic attestations of attributes; </br> (c) ‘Non_Q_EAA_Provider’ to express the entitlement of the wallet-relying party as a trust service provider issuing non-qualified electronic attestations of attributes; </br> (d) ‘PUB_EAA_Provider’ to express the entitlement of the wallet-relying party as a provider of electronic attestations of attributes issued by or on behalf of a public sector body responsible for an authentic source; </br> (e) ‘PID_Provider’ to express the entitlement of the wallet-relying party as a provider of person identification data; </br> (f) ‘QCert_for_ESeal_Provider’ to express the entitlement of the wallet-relying party as a qualified trust service provider issuing qualified certificates for electronic seals; </br> (g) ‘QCert_for_ESig_Provider’ to express the entitlement of the wallet-relying party as a qualified trust service provider issuing qualified certificates for electronic signatures; </br> (h) ‘rQSigCDs_Provider’ to express the entitlement of the wallet-relying party as a qualified trust service provider providing qualified trust services for the management of a remote qualified electronic signature creation device; </br> (i) ‘rQSealCDs_Provider’ to express the entitlement of the wallet-relying party as a qualified trust service provider providing qualified trust services for the management of a remote qualified electronic seal creation device; </br> (j) ‘ESig_ESeal_Creation_Provider’ to express the entitlement of the wallet-relying party as a non-qualified trust service provider providing a non-qualified trust service for remote creation of electronic signatures or electronic seals. | `WalletRelyingPartyService.entitlement` | [ETSI TS 119 475](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/287) defines a normative URI for each entitlement type that SHALL be used to indicate the registered entitlement. |
| 13. | Sub Entitlements | With regard to paragraph 12, point (c), Member States may provide additional sub-entitlements to state which attestations a specific non-qualified issuer of electronic attestation of attributes shall issue. | `WalletRelyingPartyService.subEntitlement` |
| 14. | Use of Intermediary | Where applicable, an indication that the wallet-relying party relies upon an intermediary acting on behalf of the relying party who intends to rely upon the wallet. | *See paragraph 15.* | Note, that the indication that the wallet-relying party relies upon an intermediary is implied by the presence of one or more `WalletRelyingPartyService.usesIntermediary` attributes. |
| 15. | Used Intermediary  | Where applicable, an association to the intermediary that the wallet-relying party is relying upon that is acting on behalf of the relying party who intends to rely upon the wallet. | `WalletRelyingPartyService.usesIntermediary` | Note, that this attribute may only be present in case of a regular wallet-relying party, not in case of an intermediary. |
| 16. | Served Wallet-Relying Party Service  | Where applicable, an association to the wallet-relying party service that is relying upon the intermediary to whom the wallet-relying party access certificate has been issued and that is acting on behalf of the relying party service who intends to rely upon the wallet. | `WalletRelyingPartyService.servedWRPS` | 1) Note, that this attribute may only be present in case of an intermediary, not in case of a regular wallet-relying party.</br></br> 2)  The requirement for this attribute has been introduced in Annex I of CIR [(EU) 2026/1730](https://eur-lex.europa.eu/eli/reg_impl/2026/1730/oj).  |
| *See Note*   | Supervisory Authority   | The competent data protection supervisory authority supervising the wallet-relying party and its intended uses. <br/>The national registry SHALL register the competent data protection supervisory authority, which is in charge of supervising the wallet-relying party, with their contact information in the mechanism offered for registering wallet-relying parties.  | `supervisoryAuthority` | 1) The legal basis for registering this information is implied by Article 5a (5) (a) (x) of [(EU) No. 910/2014](http://data.europa.eu/eli/reg/2014/910/oj) and Article 7 of [(EU) 2024/2982](http://data.europa.eu/eli/reg_impl/2024/2982/oj). </br> 2) See [TS 8](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts8-common-interface-for-reporting-of-wrp-to-dpa.md) for more information. |
| *See Note*   | Provides Attestations   | A list of attestation types the Wallet-Relying Party Service intends to issue for Wallet Units.  | `WalletRelyingPartyService.providesAttestations` | 1) The legal basis for this attribute is [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj) (ANNEX I Nr. 9)), which states that "for each intended use, a list of the data, **including attestations and attributes**" shall be registered. </br></br> 2) This attribute SHALL be registered if and only if the entitlements of the Wallet-Relying Party Service are of type 'QEAA_Provider', 'Non_Q_EAA_Provider', 'PUB_EAA_Provider' or 'PID_Provider'.|

## 4. References

| Reference | Description |
|-----------|-------------|
| [ETSI TS 119 475](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/287) | ETSI TS 119 475 V1.2.1 (2026-03): *Electronic Signatures and Trust Infrastructure (ESI); Relying Party attributes supporting EUDI Wallet user's authorisation decisions*
| [(EU) No 389/2012](http://data.europa.eu/eli/reg/2012/389/oj) | COUNCIL REGULATION (EU) No 389/2012 of 2 May 2012 on administrative cooperation in the field of excise duties and repealing Regulation (EC) No 2073/2004 |
| [(EU) No 1352/2013](http://data.europa.eu/eli/reg_impl/2013/1352/oj) | COMMISSION IMPLEMENTING REGULATION (EU) No 1352/2013 of 4 December 2013 establishing the forms provided for in Regulation (EU) No 608/2013 of the European Parliament and of the Council concerning customs enforcement of intellectual property rights |
| [(EU) No 910/2014](http://data.europa.eu/eli/reg/2014/910/oj) | REGULATION (EU) No 910/2014 OF THE EUROPEAN PARLIAMENT AND OF THE COUNCIL of 23 July 2014 on electronic identification and trust services for electronic transactions in the internal market and repealing Directive 1999/93/EC |
| [(EU) 2021/1042](http://data.europa.eu/eli/reg_impl/2021/1042/oj) | Commission Implementing Regulation (EU) 2021/1042 of 18 June 2021 laying down rules for the application of Directive (EU) 2017/1132 of the European Parliament and of the Council as regards technical specifications and procedures for the system of interconnection of registers and repealing Commission Implementing Regulation (EU) 2020/2244 (Text with EEA relevance) |
| [(EU) 2022/1860](http://data.europa.eu/eli/reg_impl/2022/1860/oj) | COMMISSION IMPLEMENTING REGULATION (EU) 2022/1860 of 10 June 2022 laying down implementing technical standards for the application of Regulation (EU) No 648/2012 of the European Parliament and of the Council with regard to the standards, formats, frequency and methods and arrangements for reporting |
| [(EU) 2024/1183](http://data.europa.eu/eli/reg/2024/1183/oj)| REGULATION (EU) 2024/1183 OF THE EUROPEAN PARLIAMENT AND OF THE COUNCIL of 11 April 2024 amending Regulation (EU) No 910/2014 as regards establishing the European Digital Identity Framework |
| [(EU) 2025/848](http://data.europa.eu/eli/reg_impl/2025/848/oj) | COMMISSION IMPLEMENTING REGULATION (EU) 2025/848 of 6 May 2025 laying down rules for the application of Regulation (EU) No 910/2014 of the European Parliament and of the Council as regards the registration of wallet-relying parties |
| [(EU) 2026/1730](http://data.europa.eu/eli/reg_impl/2026/1730/oj) | COMMISSION IMPLEMENTING REGULATION (EU) 2026/1730 of 15 July 2026 amending Implementing Regulation (EU) 2025/848 as regards applicable standards and specifications |
| [TS 2](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts2-notification-publication-provider-information.md) | European Commission: *Specification of systems enabling the notification and subsequent publication of Provider information* |
| [TS 5](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts5-common-formats-and-api-for-rp-registration-information.md)| European Commission: *Specification of common formats and API for Relying Party Registration information* |
| [TS 7](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts7-common-interface-for-data-deletion-request.md) | European Commission: *Specification of Common Interface for Data Deletion Requests to Relying Parties* |
| [TS 8](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts8-common-interface-for-reporting-of-wrp-to-dpa.md) | European Commission: *Specification of Common Interface for reporting of Relying Parties to Data Protection Authorities* |
