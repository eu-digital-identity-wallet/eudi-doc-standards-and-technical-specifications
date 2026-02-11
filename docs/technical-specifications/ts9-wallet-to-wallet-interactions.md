<img align="right" height="50" src="https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-srv-web-issuing-eudiw-py/34015dc3c6f52529a99e673df1d4fa69d50f7ff5/app/static/ic-logo.svg"/><br/>

# Specification of Wallet to Wallet interactions

## Abstract

The present document specifies the common protocols and interfaces according to Article 5a (4) (c) and Article 5a (5) (a) (vi) of
[(EU) No 910/2014](http://data.europa.eu/eli/reg/2014/910/2024-10-18) used by a Wallet Unit to interact with another
Wallet Unit.

### [GitHub Discussion](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/discussions/381)

## Licensing and Reuse

© European Union, 2025-2026.

This document is made available under the Creative Commons Attribution 4.0 International licence (CC BY 4.0), unless otherwise stated.

You may reuse this document provided that appropriate credit is given and any changes are indicated.

The full licence text is available at:
https://creativecommons.org/licenses/by/4.0/

## Versioning

| Version | Date       | Description                               |
|---------|------------|-------------------------------------------|
| `0.1`   | 2025-06-13 | Initial version for first discussions.    |
| `0.2`   | 2025-07-01 | Updated after first focus group meeting.  |
| `0.3`   | 2025-07-11 | Updated after second focus group meeting. |
| `1.0`   | 2025-07-28 | Final version after the Coordination Group review period. |
| `1.0.1` | 2026-01-30 | Editorial update (licensing and reuse clarification) |

## 1. Introduction

The present document specifies how two EUDI Wallets shall interact to exchange PID or other Attestations.

This specification is designed to enable the high level requirements defined in the ARF. Additionally the specification strives to ensure that the Wallet-to-Wallet (W2W) interaction is consistent with similar functionality elsewhere in the EUDI Wallet ecosystem. I.e., the mechanism must be compatible with ISO/IEC 18013-5. It is a goal of the specification, that the solution does not introduce unnecessary complexity.

### 1.1 W2W Use Cases (from ARF HLRs)

The [Topic J Discussion Paper](https://eu-digital-identity-wallet.github.io/eudi-doc-architecture-and-reference-framework/2.4.0/discussion-topics/j-wallet-to-wallet-interactions/) outlines a single use-case:

1. Two EUDI Wallet Users meet in physical proximity and agree (out-of-band of the EUDI Wallet system), that one (the Holder) should present a specific PID or other attestation (including the specific attributes), to the other (the Verifier).
2. Both Users select Wallet-to-Wallet mode in their respective Wallet Unit UI and are asked to specify their role (Holder or Verifier).
3. The Holder is given an option to suggest which PID or attestations (and which attributes) to present to the Verifier. This information is dubbed a "presentation offer".
4. A handshake protocol is performed and a data connection is established between the two devices, as specified in ISO/IEC 18013-5. The handshake protocol also sends the presentation offer to the Verifier, if it was specified.
5. If the Holder specified a presentation offer in Step 3, this is now displayed to the Verifier in their Wallet Unit UI. Based on this, the Verifier specifies what PID or attestation (and which attributes) will be requested from the Holder Wallet Unit, i.e. a presentation request. Note that the presentation request can be created from scratch if there was no presentation offer. If there was a presentation offer, the request can contain all or a subset of the attributes specified in the presentation offer, but no other attributes.
6. The Verifier's Wallet Unit sends the presentation request to the Holder.
7. The Holder is prompted to present the requested attributes to the Verifier, i.e., to accept the presentation request. Note that the Holder Wallet Unit will check if the presentation request matches the presentation offer created in Step 3.
8. If the Holder approves the presentation, then the Holder Wallet Unit presents the approved attributes to the Verifier Wallet Unit.
9. In the Verifier Wallet Unit, the received presentation is verified and presented in the UI.

### 1.2 Scope

This STS defines the requirements that must be supported by each Wallet Unit to enable the interactions described above. It will largely follow the ISO/IEC 18013-5 standard.
However, this document will also specify the extensions and minor deviations from ISO/IEC 18013-5 that are necessary to support W2W interactions. In particular:

1. _Technical format_ for a presentation offer and how it is _transferred_. See [2.4.2 Presentation Offer](#242-presentation-offer).
2. _Restrictions_ on creation of presentation requests. See [2.5.2 Requesting Data](#252-requesting-data).

Additionally, in [Appendix A Security Objectives and Mechanisms](#appendix-a-security-objectives-and-mechanisms), the various security objectives specific to W2W interactions are presented, as well as mechanisms for achieving these.

> Note that W2W interactions will _only_ support attestations based on the mdoc format complying with ISO/IEC 18013-5. This follows from the facts that W2W interactions only take place in proximity, and that for proximity transactions, the ARF requires the use of ISO/IEC 18013-5. This has been discussed and accepted by Member States in the beginning of the ARF drafting process. While also supporting attestation in SD-JWT VC format for W2W interactions is seen as a useful feature by some Member States, it would severely complicate this specification and imply extensive changes elsewhere in the ARF and other specifications.

### 1.3 Requirements Notation

The key words "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/74) [RFC8174](https://datatracker.ietf.org/doc/html/rfc8174) when, and only when, they are written in all capital letters.

## 2. Specification for W2W Interactions

### 2.1 Terminology

The following terminology will be used in the specification.

| Term                       | Meaning        |
|----------------------------|----------------|
| Verifier                   | An EUDI Wallet User that wishes to find out attested information about another EUDI Wallet user. |
| Holder                     | An EUDI Wallet User that wishes to share attested information with another EUDI Wallet user. |
| Presentation offer         | A description of the attested information, that a Holder wishes to share with the Verifier.  |
| Presentation request       | A description of the attested information, that the Verifier wishes the Holder to present. |
| Presentation               | The attested information being sent from the Holder to the Verifier.  |
| Wallet-Relying Party (WRP) | A party relying on the EUDI Wallets for the provision of services and therefore subject to registration requirements cf. Article 5b of [European Digital Identity Regulation] |

### 2.2 Overview

At a high-level, W2W interactions SHALL be performed following the ISO/IEC 18013-5 device retrieval method. The Holder's Wallet Unit SHALL act as a mdoc (cf. ISO/IEC 18013-5) and the Verifier's Wallet Unit SHALL act as an mdoc Reader (cf. ISO/IEC 18013-5).
The `Device Engagement` structure of ISO/IEC 18013-5 will be extended to also optionally include the Holder's presentation offer.
The presentation offer SHALL be a `NameSpaces` structure cf. ISO/IEC 18013-5:2021 p. 30.
If present, a presentation offer will impose restrictions on the Verifier's `mdoc request`, such that the `DataElements` in such subsequent mdoc request SHALL be a subset of the `DataElements` in the received presentation offer.

### 2.3 Initialization

Before any information is transferred between the Holder and Verifier, it is important that both Users actively accept that data is about to be exchanged in a W2W setting. Furthermore, the Users shall be notified about the limitations of the W2W use case compared to the regular Relying Party use case, i.e., it is only intended for natural persons and that both Users should consider if the other party is trustworthy (as discussed in [Appendix A Security Objectives and Mechanisms](#appendix-a-security-objectives-and-mechanisms)).

**STS9_01** Wallet Units SHALL offer the user an option to activate W2W and proceed to Device Engagement.

**STS9_02** Wallet Units SHALL notify the User that W2W functionality should only be used with natural person they trust, before the W2W mode is activated.

**STS9_03** A Wallet Unit SHALL NOT send or accept a Device Engagement structure containing a presentation offer before the User has activated the W2W mode.

**STS9_04** When entering W2W mode, a Wallet Unit SHALL allow the User to choose a role of either Holder or Verifier.

### 2.4 Device Engagement

**STS9_05** Device engagement for Wallet Units to do W2W interactions SHALL follow ISO/IEC 18013-5 with the extensions and restrictions described in this section.

**STS9_06**The Holder's Wallet Unit SHALL take the role of a mdoc and the Verifier's Wallet Unit SHALL take the role of a mdoc reader.

#### 2.4.1 Device Engagement Technologies

**STS9_07** A Wallet Unit SHALL support QR codes for engaging as a Holder in a W2W interaction, meaning it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc regarding the use of QR codes for device engagement.

**STS9_08** A Wallet Unit SHALL support QR codes for engaging as a Verifier in a W2W interaction, meaning it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc reader regarding the use of QR codes for device engagement.

> Note that this is a more stringent requirement than in ISO/IEC 18013-5, which solves the interoperability challenge by allowing mdocs to support either NFC or QR code for device engagement, whereas mdoc readers must support both NFC and QR code. Thus, verifiers must ensure that they use a device supporting NFC, which is reasonable given that they are legal entities that typically use dedicated devices for interacting with mdocs. In a W2W setting, on the other hand, the device of the Verifier is their personal device, and we cannot assume it supports NFC, as NFC is not supported by all User devices on the market. Therefore, it is necessary to mandate an engagement technology that works across all User devices.

**STS9_10** A Wallet Unit MAY support NFC for engaging as a Holder in a W2W interaction. If it does, it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc regarding the use of NFC for device engagement.

**STS9_11** A Wallet Unit MAY support NFC for engaging as a Verifier in a W2W interaction. If it does, it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc reader regarding the use of NFC for device engagement.

#### 2.4.2 Presentation Offer

**STS9_12** The Holder's Wallet Unit SHOULD allow the Holder to select a set of attributes available in their already issued attestations.

> Note that it is *optional* but recommended for Wallet Providers to enable the Holder to make presentation offer.

**STS9_13** The Wallet Unit SHALL authenticate the Holder before presenting the Holder with the available attributes when enabling the Holder to make such a selection.

**STS9_14** If the User makes such a selection, the Holder's Wallet Unit SHALL include them in a `PresentationOffer` under the key `-1` in the `DeviceEngagement` structure.

**STS9_15** The `DeviceEngagement` structure used for W2W interactions SHALL follow the structure below:

```
DeviceEngagement = {
	0: tstr, ; Version
	1: Security,
	? 2: DeviceRetrievalMethods, ; Is absent if NFC is used for device engagement
	? 3: ServerRetrievalMethods,
	? 4: ProtocolInfo,
	? -1: PresentationOffer
	* int => any
}
```

**STS9_16** The optional `ServerRetrievalMethods` SHALL be absent.

**STS9_17** All key-value pairs SHALL comply with applicable requirements in ISO/IEC 18013-5, except `PresentationOffer`, which SHALL be defined as:

```
PresentationOffer = {
	0: tstr, ; Version
	1: [+ DocOffer]; Documents offered
}

DocOffer = {
	0: DocType 		; Document type offered 
	1: NameSpacesOffer	; Namespaces offered
}

NameSpacesOffer : = {
	+ NameSpace => [ + DataElementIdentifier] ; Data elements offered for each namespace
}
```

**STS9_18** The key `0` of the `Presentation Offer` contains a version number that in the current version of this document SHALL be "1.0".

`DocOffer` contains the document type as a `DocType` and the offered namespaces in `NameSpacesOffer`.

`DocType` follows the ISO/IEC 18013-5:2021 definition for mdoc requests (p. 29).

`NameSpacesOffer` is a CBOR map containing NameSpaces of type `NameSpace`, and for each `NameSpace` an array of data element identifiers with type `DataElementIdentifier`.

`NameSpace` and `DataElementIdentifier` follow the ISO/IEC 18013-5:2021 definition for mdoc requests (p. 29).

> Note: Rather than letting the `PresentationOffer` mirror the `DeviceRequest` structure of ISO/IEC 18013-5:2021, it has been changed to minimize the size of the structure, since it must be transferred using QR codes.

Below is an example of a presentation offer using CBOR Diagnostic Notation (CDN)

```
{
	0: "1.0", 
	1: [
		{
			0: "org.iso.18013.5.1.mDL",
			1: {
				"org.iso.18013.5.1": [
						"family_name",
						"document_number",
						"driving_privileges",
						"issue_date",
						"expiry_date",
						"portrait",
						]
			}
		}
	]
}
```

### 2.5 Data Retrieval

**STS9_19** Data retrieval for Wallet Units to do W2W interactions SHALL follow ISO/IEC 18013-5 with the extensions and restrictions described in this section.

#### 2.5.1 Data Retrieval Transmission Technologies

**STS9_20** Wallet Units SHALL support device retrieval.

**STS9_21** Wallet Units SHALL NOT support server retrieval.

**STS9_22** A Wallet Unit SHALL support BLE for data retrieval as a Holder in a W2W interaction, meaning it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc regarding the use of  BLE for data retrieval.

**STS9_23** A Wallet Unit SHALL support BLE for data retrieval as a Verifier in a W2W interaction, meaning it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc reader regarding the use of BLE for data retrieval.

>Note that in ISO/IEC 18013-5, a mdoc must support either BLE or NFC, whereas mdoc readers must support both BLE and NFC. However, since NFC is not supported by all smartphones, it is necessary to mandate a data retrieval technology that works across all smartphones.

**STS9_24** A Wallet Unit MAY support NFC for data retrieval as a Holder in a W2W interaction. If it does, it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc regarding the use of NFC for data retrieval.

**STS9_25** A Wallet Unit MAY support NFC for data retrieval as a Verifier in a W2W interaction. If it does, it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc reader regarding the use of NFC for data retrieval.

**STS9_26** A Wallet Unit MAY support Wi-Fi Aware for data retrieval as a Holder in a W2W interaction. If it does, it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc regarding the use of Wi-Fi Aware for data retrieval.

**STS9_27** A Wallet Unit MAY support Wi-Fi Aware for data retrieval as a Verifier in a W2W interaction. If it does, it SHALL comply with all ISO/IEC 18013-5 requirements for a mdoc reader regarding the use of Wi-Fi Aware for data retrieval.

>Note that according to ISO/IEC 18013-5, an mdoc reader is recommended to (i.e., SHOULD) support Wi-Fi Aware for data retrieval.

#### 2.5.2 Requesting Data

**STS9_28** The Verifier Wallet Unit SHALL enable the Verifier to construct an mdoc request as follows:

- If a presentation offer is present in the Device Engagement information received, then the Verifier SHALL be able to select a subset (including their respective document types and name spaces) of the `DataElementIdentifiers` to include in their mdoc request.
- If a presentation offer is not present in the Device Engagement information received, then the Verifier SHALL be able to select a set of `DataElementIdentifiers` to include in their mdoc request without restrictions. The Wallet Provider SHALL decide which `DataElementIdentifiers` (including their respective document types and name spaces) the Verifier may select from to request in this case.

**STS9_29** All `IntentToRetain` flags SHALL be set to `false` in a mdoc request used in a W2W interaction.

**STS9_30** The optional `ReaderAuth` field SHALL NOT be included in an mdoc request used in a W2W interaction.
> For a discussion of this see [Appendix A Security Objectives and Mechanisms](#appendix-a-security-objectives-and-mechanisms).

#### 2.5.2.1 Rate Limiting

**STS9_31** To prevent misuse of the W2W functionality, Wallet Providers SHALL implement rate limiting for the Verifier functionality of their Wallet Units.

**STS9_32** A Wallet Provider SHALL ensure that a Wallet Unit in Verifier Wallet Unit mode cannot send more than 5 presentation requests per hour, more than 20 per day, and more than 50 per week. 

> Note that "hour", "day", and "week" in this requirement refer to sliding time periods, rather than calendar-aligned intervals. If this limit proves to be too restrictive for certain valid use cases, this limit will be revisited in future versions of this specification.

**STS9_33** A Wallet Provider MAY use increasing backoff times between subsequent presentation requests, provided these limits are not exceeded.

#### 2.5.3 Presenting Data

**STS9_34** If a presentation offer was sent as part of the W2W interaction, then the Holder's Wallet Unit, after receiving an mdoc request, SHALL validate that the `DataElementIdentifiers` in the mdoc request are a subset of the `DataElementIdentifiers` (for their respective document types and name spaces) sent in the presentation offer and that all `IntentToRetain` flags are set `false`. If not, then the Holder's Wallet Unit SHALL abort the interaction by sending a `DeviceResponse` with the field `status` set to `10` and not including any values for the optional field `documents`. In addition, the Holder Wallet Unit SHALL inform its User that the interaction was aborted because the Verifier requested attributes that were not offered by the Holder. 

> Implementation note: To prevent the Verifier from inducing information based upon the timing of the response, the time it takes for the wallet to send back this error must depend only on the data available in the `PresentationOffer` and the `DeviceRequest` (i.e., data that is already available to the Verifier's device).

> Note: An alternative approach would have been to let the Holder Wallet Unit only return errors for the specific `DataElements` that were not part of the presentation offer (rather than not returning any attributes at all). However, to prevent disclosing too much information and since this will **only** happen in cases where the Verifier Wallet Unit is not conforming to this document, this specification has opted to let the Holder's Wallet Unit return no attributes at all.

#### 2.5.4 Receiving Data

**STS9_35** When a Wallet Unit receives a mdoc response it SHALL comply with the requirements in ISO/IEC 18013-5 clause 8.3.2.1.2.1 regarding retaining data elements for which the IntentToRetain flag was set to `false`.

> Note that even though that the data itself shall not be persisted, metadata about the event must still be logged as described in HLR _DASH\_03b_ from [Annex 2 - ARF](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-2/annex-2-high-level-requirements.md#hlrs--11).

**STS9_36** Wallet Units SHOULD take measures to prevent users from taking screenshots of received presentations.

**STS9_37** A Verifier Wallet Unit SHALL NOT communicate any attribute values received from the Holder Wallet Unit to any other party inside or outside the EUDI Wallet ecosystem.

## Appendix A Security Objectives and Mechanisms

### A.1 Security Objectives

Article 5a of [European Digital Identity Regulation] specifies that each EUDI Wallet must be able to authenticate another EUDI Wallet in a W2W Interaction.
Noteworthy, the [European Digital Identity Regulation] defines an EUDI Wallet as containing a PID and authentication as "... an electronic process that enables the confirmation of a natural or legal person or the confirmation of the origin and integrity of data in electronic form" (Article 3).

In addition to the above main functional requirement, several security objectives exist for the W2W functionality:

| Enumeration   | Objective                                           | Description     |
|---------------|-----------------------------------------------------|-----------------|
| _O1_          | Authentication of the Verifier.                     | For some use cases, a Holder may wish to authenticate the Verifier to whom they are making the presentation. The requirements for the degree of certainty of authentication may vary widely from use case to use case.     |
| _O2_          | Preventing WRPs from misusing the W2W functionality. | WRPs are subject to registration requirements for relying on EUDI Wallets for presentation of attributes. It is an objective to ensure that WRPs do not use the W2W functionality to rely on presentations and thereby circumvent the registration requirements. |
| _O3_          | Prevention of data collection.                      | The W2W functionality must not enable the unnecessary collection of data. In particular, it must be made difficult for a Verifier to persist the presentation received from a Holder.  |

> Naturally, there also exists a number of requirements for the actual presentation i.e., trustworthiness of the information received in a presentation etc. However, these are no different from presentation towards WRPs and are therefore not discussed further here. However, we note in particular that checking the validity of a Holder's Wallet Unit can be done following the same approach used for this when a Wallet Unit makes a presentation to an WRP. That is, the Holder Wallet Unit is authenticated as part of the presentation flow itself. By presenting a valid Attestation, the Verifier may assume that the Attestation Provider has authenticated the Holder at the time of issuance and that the Holder was in possession of a valid Wallet Unit at the time of issuance. If the attestation presented is a valid PID credential, then the Verifier can additionally trust that they are talking to a non-revoked Wallet Unit at the time of presentation, as there is a legal requirement for PID Providers to revoke the PID credentials on a Wallet Unit if that Wallet Unit is revoked. Note that the Verifier can always request a PID presentation if this level of security is required.

> Note that even though _O2_ is a security objective, it is unclear what incentives an WRP has to pursue misusing the W2W functionality aside from the administrative burden of registering. In particular, it is unlikely that a WRP can use illegally (by the circumvention of the registration requirements) obtained data to be compliant with KYC requirements etc. Further, using it for large scale data collection is impaired by the proximity requirements for using the W2W functionality.

### A.2 Mechanisms

#### A.2.1 Enabled Mechanisms

Because of the proximity required for a Holder and a Verifier to engage in a W2W interaction,  **out-of-band** verification can be used to achieve all three objectives at least to some degree. In particular, the natural person operating the Verifier Wallet Unit will be right in front of the Holder, which already may provide certainty about the person to whom the presentation is made, (i.e. achieving _O1_), for instance when the Holder personally knows the Verifier. Further, out of band authentication, for instance by using a physical identity document, may also be performed by the Holder before agreeing to participate in a W2W interaction. Additionally, even if the Holder does not know the Verifier, the physical context may also allow the Holder to determine whether they are expecting to interact with a Relying Party or with a private natural person. Even though this does not give 100% certainty, this will allow to achieve _O2_ to some degree. Finally, as the Holder is able to physically observe the Verifier while the presentation is done, this enables the Holder to observe with a low degree of certainty if the Verifier uses out-of-band mechanisms to persist the data received in a presentation. Thereby some degree of _O3_ can be achieved.

If the use case requires it, the Holder and Verifier may opt to reverse roles of the W2W interaction to let the original Verifier do a PID presentation towards the original Holder. This means the ISO/IEC 18013-5 protocol is run twice. This will ensure that the original Holder is able to authenticate the Verifier to a high degree of certainty. Thereby, the Holder can achieve objective _O1_.

This technical specification also encompasses technical mechanisms to help achieve _O2_ and _O3_. In particular, the requirements about rate limiting makes it cumbersome for Relying Parties to use an authentic Wallet Unit, due to the limited rate of presentations. This helps towards achieving _O2_ and also makes it impossible to use an authentic Wallet Unit for large scale data collection (i.e., achieving _O3_). The requirement that a Verifier Wallet Unit shall not persist the data in a received presentation further renders it difficult to use a authentic Wallet Unit for collecting data (helping achieve _O3_). We note that, a fraudulent Relying Party may try to use a fake Wallet Unit to circumvent these. 

#### A.2.2 Possible Enhancements

To further strengthen the effectiveness of the above technical requirements, this specification could design a technical mechanism ensuring that a Holder Wallet Unit will only make a presentation towards a valid (i.e., authentic and non-revoked) Verifier Wallet Unit.
For this purpose, the ISO/IEC 18013-5 protocol provides `mdoc reader authentication`, which essentially means that the Verifier (the mdoc reader) signs some data and adds the signature in the `ReaderAuth` structure in the `mdoc request`.
However, this signature is based on the use of X.509 certificates. In the EUDI Wallet ecosystem as current designed in the ARF, a Verifier Wallet Unit does not have such a certificate. (In contrast, a WRP has a X.509 access certificate, which it receives during registration.)
Therefore, to authenticate a Verifier Wallet Unit, two options exist:

1. Modify the mechanism to be compatible with the Wallet App Attestation (WAA) from [TS3 Wallet Unit Attestations](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts3-wallet-unit-attestation.md), which is a JWT issued by the Wallet Provider. While the WAA may be mapped to CBOR (i.e. to a CWT) as specified in [CBOR Web Token](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/413), some changes to the `ReaderAuth` functionality would be needed. This will make ISO/IEC 18013-5 compliance much harder to achieve and would (ideally) require changes to the standard, which would be a fairly slow process. Note further, that the WAA is ephemeral or has an expiration time of less than 24 hours, meaning that imposing such mechanism would render the W2W functionality impossible to use offline.

> In fact, to achieve a high degree of certainty of talking to a Wallet Unit it is necessary to let such certificate/attestation have a short time to live.

2. The Wallet Provider could also issue an X.509 authentication certificate to each Wallet Unit directly, in which case ISO/IEC 18013-5 `ReaderAuth` can be used unmodified. While this would achieve compliance to the standard, it would put an additional burden on Wallet Providers and add technical complexity to the ecosystem, as Wallet Providers would now also have to issue and handle X.509 certificates in addition to the existing JWT-based WUAs and WAAs.

> Note that even with such envisioned hardening _O2_ and _O3_ will not be fully guaranteed. In particular, it will always be technically possible for an WRP to misuse the W2W functionality to receive presentations - only the scale and usefulness of this can be reduced. Moreover, a Verifier may extract data from a presentation using out-of-band mechanisms, for instance by writing notes or taking photos of the presentation. In the extreme, a Verifier may simply remember the received presentation and persist it when not observed.

Based on the above discussion of security objectives and methods for reaching these, this specification opts to let out-of-band mechanisms ensure a baseline of _O1_, _O2_, and _O3_. Further, the specification encompasses requirements for Verifier Wallet Units on rate limiting and non-persistence of presented attributes to achieve _O2_ and _O3_. Finally, in high-assurance cases a reversed W2W interaction can be used to obtain _O1_ with high certainty. We deem that the additional complexity for the ecosystem by hardening _O2_ and _O3_ with verifying the validity of Wallet Units outweighs its benefits and no such mechanism is therefore included in the specification.
