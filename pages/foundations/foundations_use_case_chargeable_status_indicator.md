---
title: Visitors and Migrants Chargeable-Status Indicator
keywords: nhsnumber, chargeable-status
tags: [foundations,use_case]
sidebar: foundations_sidebar
permalink: foundations_use_case_chargeable_status_indicator.html
summary: "Use case to search for an overseas Visitors and Migrants chargeable status indicator."
---

## Prerequisites ##

To use this API, the requester:

- SHALL have gone through accreditation and received an endpoint certificate and associated ASID (Accredited System ID) for the client system.
- SHALL have either:
	- Authenticated the user using national smartcard authentication, and obtained a the UUID from the user's smartcard (and associated RBAC role from CIS), or
	- Authenticated the user using an assured local mechanism, and obtained a local user ID and role
	- And pass this user information in a JSON web token - see [Cross Organisation Audit and Provenance](integration_cross_organisation_audit_and_provenance.html) for details.
- SHALL have previously traced the patient's NHS Number using PDS or an equivalent service.

## Request Headers ##

All Visitors and Migrants APIs should include the below additional HTTP request headers to support audit and security requirements on the Spine:

| Header               | Value |
|----------------------|-------|
| `Ssp-TraceID`        | Client System TraceID (i.e. GUID/UUID). This is a unique ID that the client system should provide. It can be used to identify specific requests when troubleshooting issues with API calls. All calls into the service should have a unique TraceID so they can be uniquely identified later if required. |
| `Ssp-From`           | Client System ASID |
| `Ssp-To`             | The Spine ASID |
| `Ssp-InteractionID`  | `urn:nhs:names:services:visitorsandmigrants:fhir:rest:search:observation`|
| `Ssp-Version`  | `1` |
| `Authorization`      | This will carry the base64 encoded JSON web token required for audit - see [Cross Organisation Audit and Provenance](integration_cross_organisation_audit_and_provenance.html) for details. |

- Note: The Ssp-Version defaults to 1 if not supplied (this is currently the only version of the API). This indicates the major version of the interaction, so when new major releases of this specification are released (for example releases with breaking changes), implementors will need to specify the correct version in this header.

## Search for Chargeable-Status Indicator ##

Search for a Visitors and Migrants Chargeable-Status Indicator `Observation` for a specified patient using a business identifier (i.e. the NHS Number).

```http
GET [base]/Observation?subject:Patient.identifier=[system]|[value]&code=[system]|[code]
```

- The `[system]` field for the Paient Itentifier SHALL be populated with the identifier system URL: `https://fhir.nhs.uk/Id/nhs-number`.
- The `[system]` field for the Code SHALL be populated with the identifier system URL: `https://fhir.nhs.uk/fhir-observation-code-1`.
- The `[code]` will be taken from the [fhir-observation-code-1 valueset](https://fhir.nhs.uk/ValueSet/fhir-observation-code-1) and for this API will always be 0001.
- The `[base]` is the URL of the Spine endpoint. Note: Details of Spine endpoint addresses for test regions can be found on the [Assurance Support Portal](http://www.assurancesupport.digital.nhs.uk/)
- Note: The mime-type can be specified to request either XML or JSON using another URL parameter `?_format=[mime-type]`, or a `Content-Type` HTTP header as per the [FHIR specification](https://www.hl7.org/fhir/http.html#mime-type).

So, an example GET request for NHS number 1234567890 could be:

```http
GET https://spine-url/Observation?subject:Patient.identifier=https://fhir.nhs.uk/Id/nhs-number|1234567890&code=https://fhir.nhs.uk/fhir-observation-code-1|0001
```

NOTE: This would need to be URL encoded, so the \| characters would be encoded in the actual HTTP request sent.

### Search Response ###

Success:

- SHALL return a `200` **OK** HTTP status code on successful execution of the interaction.
- SHALL return a `Bundle` of `type` searchset, containing either:
	- One matching `Observation` resource that conforms to the `spine-vm-observation-1` profile; or
	- One `OperationOutcome` resource if the interaction is a success, however no Overseas Visitors and Migrants record has been found.
- Where an Observation is returned, it SHALL include the `versionId` and `fullUrl` of the current version of the `observation` resource.


```json
{
 "resourceType": "Bundle",
  "id": "6f759a10-d1b6-11e6-9598-0800200c9a66",
  "type": "searchset",
  "entry": [
    {
      "resource": {
        "resourceType": "Observation",
        "id": "76e39290-d1aa-11e6-9598-0800200c9a66",
        "meta": {
          "profile": [
            "https://fhir.nhs.uk/StructureDefinition/spine-vm-observation-1"
          ]
        },
        "status": "final",
        "code": {
          "coding": [
            {
              "system": "https://fhir.nhs.uk/fhir-observation-code-1",
              "code": "0001",
              "display": "Visitors and Migrants status observation"
            }
          ]
        },
        "subject": {
          "reference": "Patient/9900002831",
          "display": "TAYLOR, Mary (Miss)"
        },
        "effectiveDateTime": "2015-01-01T15:00:00+00:00",
        "component": [
          {
            "code": {
              "coding": [
                {
                  "system": "https://fhir.nhs.uk/spine-vm-observation-component-1",
                  "code": "BASIC_CHARGEABLE_STATUS",
                  "display": "Basic Chargeable Status"
                }
              ]
            },
            "valueCodeableConcept": {
              "coding": [
                {
                  "system": "https://fhir.nhs.uk/spine-chargeable-status-1",
                  "code": "Y",
                  "display": "Chargeable"
                }
              ]
            }
          },
          {
            "code": {
              "coding": [
                {
                  "system": "https://fhir.nhs.uk/spine-vm-observation-component-1",
                  "code": "CATEGORY_CHARGEABLE_STATUS",
                  "display": "Category Chargeable Status"
                }
              ]
            },
            "valueCodeableConcept": {
              "coding": [
                {
                  "system": "https://fhir.nhs.uk/spine-category-status-1",
                  "code": "F",
                  "display": "Chargeable non-EEA"
                }
              ]
            }
          }
        ]
      }
    }
  ]
}
```

- Additional examples are available here - [XML](Examples/Bundle-Observation.xml) and [JSON](Examples/Bundle-Observation.json)

Failure: 

- SHALL return one of the below HTTP status error codes with an `OperationOutcome` resource that conforms to the `spine-operationoutcome-1` profile if the search cannot be executed (not that there is no match).
- The below table summarises the types of error that could occur, and the HTTP response codes, along with the values to expect in the `ObservationOutcome` in the response body.

| HTTP Code | issue-severity | issue-type | Details.Code | Details.Display |
|-----------|----------------|------------|--------------|-----------------|
|404|error|not-found|PATIENT_NOT_FOUND|Patient not found|
|400|error|invalid|INVALID_NHS_NUMBER|Invalid NHS number|
|400|error|code-invalid|INVALID_CODE_SYSTEM|Invalid code system|
|400|error|code-invalid|INVALID_CODE_VALUE|Invalid code value|
|400|error|code-invalid|INVALID_IDENTIFIER_SYSTEM|Invalid identifier system|
|400|error|value|INVALID_ELEMENT|Invalid element|
|400|error|invalid|INVALID_PARAMETER|Invalid parameter|
|400|error|unknown|REQUEST_UNMATCHED|Request does not match authorisation token|
|400|error|structure|MESSAGE_NOT_WELL_FORMED|Message not well formed|
|400|error|invalid|MISSING_OR_INVALID_HEADER|There is a required header missing or invalid|
|403|error|forbidden|ACCESS_DENIED_SSL|SSL Protocol or Cipher requirements not met|
|403|error|forbidden|ASID_CHECK_FAILED|The sender or receiver's ASID is not authorised for this interaction|


- The error codes (including other Spine error codes that are outside the scope of this API) are defined in the [Spine Error or Warning Code ValueSet](https://fhir.nhs.uk/ValueSet/spine-error-or-warning-code-1)
- Error REQUEST_UNMATCHED would occur if the NHS number being requested in the search request does not match the requested_record value in the JWT - see [Cross Organisation Audit and Provenance](integration_cross_organisation_audit_and_provenance.html) for details.

```json
{
 "resourceType": "Bundle",
  "id": "67883730-d1a8-11e6-9598-0800200c9a66",
  "type": "searchset",
  "entry": [
    {
      "resource": {
        "resourceType": "OperationOutcome",
        "id": "ff00d600-d1a6-11e6-9598-0800200c9a66",
        "meta": {
          "profile": [
            "https://fhir.nhs.uk/StructureDefinition/spine-operationoutcome-1"
          ]
        },
        "issue": [
          {
            "severity": "error",
            "code": "invalid",
            "details": {
              "coding": [
                {
                  "system": "https://fhir.nhs.uk/spine-error-or-warning-code-1",
                  "code": "INVALID_NHS_NUMBER",
                  "display": "Invalid NHS number"
                }
              ]
            },
            "diagnostics": "An invalid NHS number format has been provided in the request"
          }
        ]
      }
    }
  ]
}
```

- Additional examples are available here - [XML](Examples/Bundle-OperationOutcome.xml) and [JSON](Examples/Bundle-OperationOutcome.json)

### Example Code ###

#### C# ####

```csharp
var client = new FhirClient("http://spine-base-url/fhir-base/");
client.PreferredFormat = ResourceFormat.Json;
var query = new string[] { "identifier=https://fhir.nhs.uk/Id/nhs-number|9900002831" };
var bundle = client.Search("Patient", query);
FhirSerializer.SerializeResourceToXml(bundle).Dump();
```

#### Java ####

```java
FhirContext ctx = new FhirContext();
IGenericClient client = ctx.newRestfulGenericClient("http://spine-base-url/fhir-base/");
Bundle bundle = client.search().forResource(Observation.class)
.where(new TokenClientParam("identifier").exactly().systemAndCode("https://fhir.nhs.uk/Id/nhs-number", "9900002831"))
.encodedXml()
.execute();
```

#### cURL ####

{% include embedcurl.html title="Search for Chargeable-Status Indicator" command="curl -X GET -H 'Ssp-From: 0001' -H 'Ssp-To: 0002' -H 'Ssp-InteractionID: urn:nhs:names:services:visitorsandmigrants:fhir:rest:search:observation' -H 'Cache-Control: no-cache' -H 'Ssp-TraceID: e623b4de-f6bb-be0c-956d-c4ded0d58fc0' 'http://spine-base-url/fhir-base/Observation?identifier=https://fhir.nhs.uk/Id/nhs-number%7C9900002831'" %}

