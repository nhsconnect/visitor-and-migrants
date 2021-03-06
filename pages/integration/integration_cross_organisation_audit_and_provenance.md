---
title: Cross Organisation Audit & Provenance
keywords: spine, ssp, integration, audit, provenance
tags: [integration]
sidebar: overview_sidebar
permalink: integration_cross_organisation_audit_and_provenance.html
summary: "Overview of how audit and provenance data is expected to be transported over the Visitors and Migrants FHIR interfaces."
---

## Cross Organisation Audit & Provenance ##

Consumer systems SHALL provided audit and provenance details in the HTTP authorization header as an oAuth bearer token (as outlined in [RFC 6749](https://tools.ietf.org/html/rfc6749){:target="_blank"}) in the form of a JSON Web Token (JWT) as defined in [RFC 7519](https://tools.ietf.org/html/rfc7519){:target="_blank"}.

An example such an HTTP header is given below:

```
     Authorization: Bearer jwt_token_string
```


In future, national authentication and authorisation services will be made available which will issue a bearer token which can be used directly for accessing this API. In the interrim however, the client will need to construct the JWT themselves.

It is highly recommended that standard libraries are used for creating the JWT as constructing and encoding the token manually may lead to issues with parsing the token in Spine. A good source of information about JWT and libraries to use can be found on the [JWT.io site](https://jwt.io/)

### JSON Web Tokens (JWT) ###

Consumer system SHALL generate a new JWT for each API request. The Payload section of the JWT (see "JWT Generation" below for futher details) shall be populated as follows:

| Claim | Priority | Description | Fixed Value | Dynamic Value | Specification / Example |
|-------|----------|-------------|-------------|---------------|-------------------------|
| iss | R | Client systems issuer URI | No | Yes | |
| sub | R | ID for the user on whose behalf this request is being made. Matches `requesting_practitioner .identifier.value` | No | Yes | |
| aud | R | Authorization server's `token_URL` | `https://authorize .fhir.nhs.uk/token` | No | |
| exp | R | Expiration time integer after which this authorization MUST be considered invalid. | `exp` | (now + 5 minutes) UTC time in seconds | |
| iat | R | The UTC time the JWT was created by the requesting system | `iat` | now UTC time in seconds | |
| reason_for_request | R | Purpose for which access is being requested | `directcare` | No | |
| requested_record | R | The FHIR patient resource being requested (i.e. NHS Number identifier details) | No | FHIR Patient<sup>1</sup> | Audit-Patient-1 [(Rendered)](https://fhir.nhs.uk/StructureDefinition/audit-patient-1) [(json)](Audit/StructureDefinitions/Audit-Patient-1.json) [(Example)](Audit/Examples/Patient.json) |
| requested_scopes | R | Data being requested | `patient/Observation.read` | No | |
| requesting_device | R | FHIR device resource making the request | No | FHIR Device<sup>1</sup> | Audit-Device-1 [(Rendered)](https://fhir.nhs.uk/StructureDefinition/audit-device-1) [(json)](Audit/StructureDefinitions/Audit-Device-1.json) [(Example)](Audit/Examples/Device.json) |
| requesting_organization | R | FHIR organisation resource making the request | No | FHIR Organization<sup>1+3</sup> | Audit-Organization-1 [(Rendered)](https://fhir.nhs.uk/StructureDefinition/audit-organization-1) [(json)](Audit/StructureDefinitions/Audit-Organization-1.json) [(Example)](Audit/Examples/Organization.json) |
| requesting_practitioner | R | FHIR practitioner resource making the request | No | FHIR Practitioner<sup>2+3</sup> | Audit-Practitioner-1 [(Rendered)](https://fhir.nhs.uk/StructureDefinition/audit-practitioner-1) [(json)](Audit/StructureDefinitions/Audit-Practitioner-1.json) [(Example - smartcard)](Audit/Examples/Practitioner1a.json) <br /> [(Example - non-smartcard)](Audit/Examples/Practitioner1b.json)|

<sup>1</sup> Minimal FHIR resource to include any relevant business identifier(s).

<sup>2</sup> To contain the practitioners SDS Role profile ID (taken from their SAML assertion, linked to their Smartcard session).

<sup>3</sup>The requesting organisation resource SHALL refer to the care organisation from where the request originates rather than any other organisation which may host hardware or software, route requests to Spine, and/or hold the endpoint registration.

{% include important.html content="In topologies where consumer applications are provisioned via a portal or middleware hosted by another organisation (e.g. a 'mini service' provider) it is important for audit purposes that the practitioner and organisation populated in the JWT reflect the originating organisation rather than the hosting organisation." %}

#### JWT Generation ####
Consumer systems SHALL generate the JSON Web Token (JWT) consisting of three parts seperated by dots (.), which are:

- Header
- Payload
- Signature

The Spine does not currently validate the signature in the JWT that is sent, so the Consumer systems MAY generate an Unsecured JSON Web Token (JWT) using the "none" algorithm parameter in the header to indicate that no digital signature or MAC has been performed (please refer to section 6 of [RFC 7519](https://tools.ietf.org/html/rfc7519){:target="_blank"} for details).

```json
{
  "alg": "none",
  "typ": "JWT"
}
```

If using unsigned tokens, the consumer systems SHALL generate an empty signature.

The final output is three Base64 strings separated by dots (note - there is some canonicalisation done to the JSON before it is base64 encoded, which the JWT code libraries will do for you).

For example:

```shell
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJpc3MiOiJodHRwOi8vZWMyLTU0LTE5NC0xMDktMTg0LmV1LXdlc3QtMS5jb21wdXRlLmFtYXpvbmF3cy5jb20vIy9zZWFyY2giLCJzdWIiOiIxIiwiYXVkIjoiaHR0cHM6Ly9hdXRob3JpemUuZmhpci5uaHMubmV0L3Rva2VuIiwiZXhwIjoxNDgxMjUyMjc1LCJpYXQiOjE0ODA5NTIyNzUsInJlYXNvbl9mb3JfcmVxdWVzdCI6ImRpcmVjdGNhcmUiLCJyZXF1ZXN0ZWRfcmVjb3JkIjp7InJlc291cmNlVHlwZSI6IlBhdGllbnQiLCJpZGVudGlmaWVyIjpbeyJzeXN0ZW0iOiJodHRwOi8vZmhpci5uaHMubmV0L0lkL25ocy1udW1iZXIiLCJ2YWx1ZSI6IjkwMDAwMDAwMzMifV19LCJyZXF1ZXN0ZWRfc2NvcGUiOiJwYXRpZW50LyoucmVhZCIsInJlcXVlc3RpbmdfZGV2aWNlIjp7InJlc291cmNlVHlwZSI6IkRldmljZSIsImlkIjoiMSIsImlkZW50aWZpZXIiOlt7InN5c3RlbSI6IldlYiBJbnRlcmZhY2UiLCJ2YWx1ZSI6IkdQIENvbm5lY3QgRGVtb25zdHJhdG9yIn1dLCJtb2RlbCI6IkRlbW9uc3RyYXRvciIsInZlcnNpb24iOiIxLjAifSwicmVxdWVzdGluZ19vcmdhbml6YXRpb24iOnsicmVzb3VyY2VUeXBlIjoiT3JnYW5pemF0aW9uIiwiaWQiOiIxIiwiaWRlbnRpZmllciI6W3sic3lzdGVtIjoiaHR0cDovL2ZoaXIubmhzLm5ldC9JZC9vZHMtb3JnYW5pemF0aW9uLWNvZGUiLCJ2YWx1ZSI6IltPRFNDb2RlXSJ9XSwibmFtZSI6IkdQIENvbm5lY3QgRGVtb25zdHJhdG9yIn0sInJlcXVlc3RpbmdfcHJhY3RpdGlvbmVyIjp7InJlc291cmNlVHlwZSI6IlByYWN0aXRpb25lciIsImlkIjoiMSIsImlkZW50aWZpZXIiOlt7InN5c3RlbSI6Imh0dHA6Ly9maGlyLm5ocy5uZXQvc2RzLXVzZXItaWQiLCJ2YWx1ZSI6IkcxMzU3OTEzNSJ9LHsic3lzdGVtIjoibG9jYWxTeXN0ZW0iLCJ2YWx1ZSI6IjEifV0sIm5hbWUiOnsiZmFtaWx5IjpbIkRlbW9uc3RyYXRvciJdLCJnaXZlbiI6WyJHUENvbm5lY3QiXSwicHJlZml4IjpbIk1yIl19fX0.
```

NOTE: As this is an unsigned token, the final section (the signature) is empty, so the JWT will end with a trailing . - this must not be omitted as it will then not be a valid token.

{% include tip.html content="The [JWT.io](https://jwt.io/) website includes a number of rich resources to aid in developing JWT enabled applications." %}

## JWT Payload Example ##

```json
{
	"iss": "https://[ConsumerSystemURL]",
	"sub": "[PractitionerID]",
	"aud": "https://authorize.fhir.nhs.uk/token",
	"exp": 1469436987,
	"iat": 1469436687,
	"reason_for_request": "directcare",
	"requested_record": {
		"resourceType": "Patient",
		"identifier": [{
			"system": "https://fhir.nhs.uk/Id/nhs-number",
			"value": "[NHSNumber]"
		}]
	},
	"requested_scopes": "patient/*.read",
	"requesting_device": {
		"resourceType": "Device",
		"id": "[DeviceID]",
		"identifier": [{
			"system": "[DeviceSystem]",
			"value": "[DeviceID]"
		}],
		"type": "[SNOMEDCTCodeForTypeOfDevice]",
		"model": "[SoftwareName]",
		"version": "[SoftwareVersion]"
	},
	"requesting_organization": {
		"resourceType": "Organization",
		"id": "[OrganizationID]",
		"identifier": [{
			"system": "https://fhir.nhs.uk/Id/ods-organization-code",
			"value": "[ODSCode]"
		}],
		"name": "Requesting Organisation Name"
	},
	"requesting_practitioner": {
		"resourceType": "Practitioner",
		"id": "[PractitionerID]",
		"identifier": [{
			"system": "https://fhir.nhs.uk/sds-role-profile-id",
			"value": "[SDSRoleProfileID]"
		}]
	}
}
```

{% include important.html content="Whilst the use of a JWT and the claims naming is inspired by the [SMART on FHIR](https://github.com/smart-on-fhir/smart-on-fhir.github.io/wiki/cross-organizational-auth) NHS Digital hasn't commited to using the SMART on FHIR specification." %}

### Example Code ###

#### C# ####

{% include tip.html content="The following code snippet utilise the [Microsoft Identity Model JWT Token Nuget Package](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt/) for creating, serializing and validating JWT tokens." %}

{% gist michaelmeasures/d6a75e52acdbee93c4c30d23e639fb1a %}

