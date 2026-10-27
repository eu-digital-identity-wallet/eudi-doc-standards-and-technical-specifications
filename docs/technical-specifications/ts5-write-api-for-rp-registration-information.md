<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification of write API methods for Relying Party Registration information

## Abstract

The present document collects currently non-normative POST, PUT and DELETE methods in application programming interface (API) for the machine-readable Relying Party Registration required by the [European Digital Identity Regulation (EU 910/2014)](https://eur-lex.europa.eu/eli/reg/2014/910/oj/eng). The document is separated from the [Specification of common formats and API for relying-party registration information](ts5-write-api-for-rp-registration-information.md) that specifies the full data model and common read-only (GET) methods for the Relying Party Registration information.

## Licensing and Reuse

© European Union, 2026.

This document is made available under the Creative Commons Attribution 4.0 International licence (CC BY 4.0), unless otherwise stated.

You may reuse this document provided that appropriate credit is given and any changes are indicated.

The full licence text is available at:
<https://creativecommons.org/licenses/by/4.0/>

## Versioning

| Version | Date        | Description                                               |
|---------|-------------|-----------------------------------------------------------|
| `0.1`   | 25.09.2026  | Initial version - write API sections separated from main TS5 specification          |


## 1 Introduction and Overview

The present document lists the non read-only methods of the common application programming interface (API) related to **Relying Party Registration** according to [European Digital Identity Regulation (EU 910/2014)](https://eur-lex.europa.eu/eli/reg/2014/910/oj/eng) - herein [Regulation] and the Commission Implementing Regulation (CIR) [EU 2026/1730](http://data.europa.eu/eli/reg_impl/2026/1730/oj) of 15 July 2026 amending Implementing Regulation (EU) 2025/848 [CIR for Relying Party Registration] as regards applicable standards and specifications.

The applicable data model and the common read-only API for Relying Party Registration is defined in [Specification of common formats and API for relying-party registration information](ts5-write-api-for-rp-registration-information.md)

## 2 Restricted-access Application Programming Interface

### 2.1 API methods on registration and updating of Wallet-Relying Party data

#### 2.1.1 Requirements

The common Registry API write methods are defined for purposes of managing the register information of Member State Registrars, and SHALL be accessible by authorised users only.

#### 2.1.2 Method description

path:
`/wrp` (POST, PUT and DELETE):

**POST** is for creating a new WalletRelyingParty. Method expects a request body with the WalletRelyingParty schema, and returns a `201` on success. If an identifier collision is detected upon intended creation, a `409` is returned.

**PUT** is for updating an existing WalletRelyingParty. Method expects a request body with the WalletRelyingParty schema, and can return `200` on success or `404` if not found.

**DELETE** is for deleting of an existing WalletRelyingParty. Method expects WalletRelyingParty identifier as a parameter, and returns a `204` on success.

`/wrp/{identifier}/services` (POST):

**POST** is for creating a new WalletRelyingPartyService. Method expects a request body with the WalletRelyingPartyService schema, and returns a `201` on success.

> Note: POST is issued directly to the parent collection to create a sub-resource, rather than supplying the `serviceidentifier` in the URL explicitly, as it is handled internally by the payload during resource creation.

If a service identifier collision (identifier exists already) is detected upon intended creation, a `409` is returned.

`/wrp/{identifier}/services/{serviceidentifier}:` (PUT and DELETE):

**PUT** is for updating an existing WalletRelyingPartyService. Method expects a request body with the WalletRelyingParty schema, and can return `200` on success or `404` if not found.

**DELETE** is for deleting of an existing WalletRelyingPartyService. Method expects WalletRelyingPartyService service identifier as a parameter, and returns a `204` on success.

> Note: **The harmonisation of the POST methods (means to accomplish registration of a new Wallet-Relying Party or once registered, management of its underlying Wallet-Relying Party Services) have been left for further study.** A basic method for creating a WalletRelyingParty instance at the Registrar is provided in the API specification in a commented-out section. The Registrar implementor should carefully consider the design options with write methods - some approach that works for large organisations with multiple actively managed (updated) services may be not suitable for Wallet-Relying Parties with just a single, stable service.

#### 3.1.3 Authentication and authorisation

The write methods SHALL be only accessible for authenticated and authorised API clients, and methods shall apply only under the authorised scope of the API client. **The national mechanisms to implement authentication and authorisation are left for the discretion of the Member States.**

The applicable mechanism SHALL be documented by adding a `securitySchemes` section in `components` and a `security` section at the root level or at the path/operation level in the individual instance of the [Annex A.1](#a1-openapi-specification-non-normative) specification.

## 3 References

| Reference                              | Description                                  |
|----------------------------------------|----------------------------------------------|
| [Regulation]                           | [European Digital Identity Regulation (EU 910/2014)](https://eur-lex.europa.eu/eli/reg/2014/910/oj/eng)  |
| [CIR for Relying Party Registration]   | [Commission Implementing Regulation (EU) 2025/848 of 6 May 2025 laying down rules for the application of Regulation (EU) No 910/2014 of the European Parliament and of the Council as regards the registration of wallet-relying parties](http://data.europa.eu/eli/reg_impl/2025/848/oj)  |
| [CIR 2026/1730] | [Commission Implementing Regulation (EU) 2026/1730 of 15 July 2026 amending Implementing Regulation (EU) 2025/848 as regards applicable standards and specifications](http://data.europa.eu/eli/reg_impl/2026/1730/oj) |
| [Specification of common formats and API for relying-party registration information] | [The European Commission Specification of common formats and API for Relying Party Registration information](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts7-common-interface-for-data-deletion-request.md) |

## Annex A

### A.1 OpenAPI Specification (non-normative)

The YAML file [`ts5-openapi31-write-api.yml`](api/ts5-openapi31-write-api.yml) contains the [OpenAPI](https://spec.openapis.org/oas/latest.html) specification of the JSON and REST based application programming interfaces described in [Section 2](#2-api-write-methods).
