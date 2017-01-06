---
title: Visitors and Migrants Chargeable-Status Indicator
keywords: nhsnumber, chargeable-status
tags: [foundations,use_case]
sidebar: foundations_sidebar
permalink: foundations_use_case_chargeable_status_indicator.html
summary: "Use case to search for and read an overseas Visitors and Migrants chargeable status indicator."
---

## Prerequisites ##

To use this API, the requester:

- SHALL have gone through accreditation and received an endpoint certificate and associated ASID (Accredited System ID) for the client system.
- SHALL have authenticated the user using national smartcard authentication, and obtained a ticket and the UUID from the user's smartcard (this is pass in a JSON web token - see [Cross Organisation Audit and Provenance](integration_cross_organisation_audit_and_provenance.html) for details)
- SHALL have previously traced the patient's NHS Number using PDS or an equivalent service.

## Request Headers ##

All Visitors and Migrants APIs should include the below additional HTTP request headers to support audit and security requirements on the Spine:

| Header               | Value |
|----------------------|-------|
| `Ssp-TraceID`        | Client System TraceID (i.e. GUID/UUID) |
| `Ssp-From`           | Client System ASID |
| `Ssp-To`             | The Spine ASID |
| `Ssp-InteractionID`  | `urn:nhs:names:services:visitorsandmigrants:fhir:rest:search:observation`|

This is in addition to the JSON web token required for audit - see [Cross Organisation Audit and Provenance](integration_cross_organisation_audit_and_provenance.html) for details.

[The above to be confirmed with the Spine team]


## Search for Chargeable-Status Indicator ##

Search for a Visitors and Migrants Chargeable-Status Indicator `Observation` for a specified patient using a business identifier (i.e. the NHS Number).

```http
GET [base]/Observation?subject:Patient.identifier=[system]|[value]&code=[system]|[code]
```

- The `[system]` field for the Paient Itentifier SHALL be populated with the identifier system URL: `http://fhir.nhs.net/Id/nhs-number`.
- The `[system]` field for the Code SHALL be populated with the identifier system URL: `http://fhir.nhs.net/fhir-observation-code-1`.
- The `[base]` is the URL of the Spine endpoint [TO BE CONFIRMED].
- Note: The mime-type can be specified to request either XML or JSON using another URL parameter `?_format=[mime-type]`, or a `Content-Type` HTTP header as per the [FHIR specification](https://www.hl7.org/fhir/http.html#mime-type).

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
  "meta": {
    "profile": [
      "http://fhir.nhs.net/StructureDefinition/spine-searchset-bundle-1"
    ]
  },
  "type": "searchset",
  "entry": [
    {
      "resource": {
        "resourceType": "Observation",
        "id": "76e39290-d1aa-11e6-9598-0800200c9a66",
        "meta": {
          "profile": [
            "http://fhir.nhs.net/StructureDefinition/spine-vm-observation-1"
          ]
        },
        "status": "final",
        "code": {
          "coding": [
            {
              "system": "http://fhir.nhs.net/fhir-observation-code-1",
              "code": "0001",
              "display": "Visitors and Migrants status observation"
            }]
        },
        "subject": {
          "reference": "Patient/e88cdcd0-d1aa-11e6-9598-0800200c9a66",
          "display": "Miss Mary Taylor"
        },
        "effectiveDateTime": "2015-01-01T15:00:00+00:00",
        "issued": "2015-02-01T10:00:00+00:00",
        "component": [
          {
            "code": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-vm-observation-component-1",
                  "code": "bcs",
                  "display": "Basic Chargeable Status"
                }]
            },
            "valueCodeableConcept": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-chargeable-status-1",
                  "code": "y",
                  "display": "Chargeable"
                }]
            }
          },
          {
            "code": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-vm-observation-component-1",
                  "code": "ccs",
                  "display": "Category Chargeable Status"
                }
              ]
            },
            "valueCodeableConcept": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-category-status-1",
                  "code": "F",
                  "display": "F"
                }]
            }
          }
        ]
      }
    }
  ]
}
```

Failure: 

- SHALL return a `4xx` or `5xx` HTTP status code  with an `OperationOutcome` resource that conforms to the `spine-operationoutcome-1` profile if the search cannot be executed (not that there is no match):
- The types of error in an `ObservationOutcome` are defined in the [Spine Error or Warning Code ValueSet](/ValueSets/spine-error-or-warning-code-1.xml)

```json
{
"resourceType": "Bundle",
  "id": "67883730-d1a8-11e6-9598-0800200c9a66",
  "meta": {
    "profile": [
      "http://fhir.nhs.net/StructureDefinition/spine-searchset-bundle-1"
    ]
  },
  "type": "searchset",
  "entry": [
    {
      "resource": {
        "resourceType": "OperationOutcome",
        "id": "ff00d600-d1a6-11e6-9598-0800200c9a66",
        "meta": {
          "profile": [
            "http://fhir.nhs.net/StructureDefinition/spine-operationoutcome-1"
          ]
        },
        "issue": [
          {
            "severity": "error",
            "code": "invalid",
            "details": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-error-or-warning-code-1",
                  "code": "VM-0002"
                }]
            },
            "diagnostics": "Invalid NHS Number"
          }]
      }
    }
  ]
}
```

### Example Code ###

#### C# ####

```csharp
var client = new FhirClient("http://gpconnect.fhir.nhs.net/fhir/");
client.PreferredFormat = ResourceFormat.Json;
var query = new string[] { "identifier=http://fhir.nhs.net/Id/nhs-number|P002" };
var bundle = client.Search("Patient", query);
FhirSerializer.SerializeResourceToXml(bundle).Dump();
```

#### Java ####

```java
FhirContext ctx = new FhirContext();
IGenericClient client = ctx.newRestfulGenericClient("http://gpconnect.fhir.nhs.net/fhir/");
Bundle bundle = client.search().forResource(Patient.class)
.where(new TokenClientParam("identifier").exactly().systemAndCode("http://fhir.nhs.net/Id/nhs-number", "P002"))
.encodedXml()
.execute();
```

#### cURL ####

{% include embedcurl.html title="Find a patient" command="curl -X GET -H 'Ssp-From: 0001' -H 'Ssp-To: 0002' -H 'Ssp-InteractionID: urn:nhs:names:services:gpconnect:fhir:rest:search:patient' -H 'Cache-Control: no-cache' -H 'Ssp-TraceID: e623b4de-f6bb-be0c-956d-c4ded0d58fc0' 'http://gpconnect.fhir.nhs.net/fhir/Patient?identifier=http://fhir.nhs.net/Id/nhs-number%7CP002'" %}

## Read Chargeable-Status Indicator ##

Read the latest Visitors and Migrants Chargeable-Status Indicator `Observation` with a specific [resource identifier](https://www.hl7.org/fhir/DSTU2/resource.html#id).

In cases where the client already has an identifier for the Visitors and Migrants chargeable status indicator Observation (for example from the results of a previous search), the Observation can be read directly using the ID:

```http
GET /Observation/[id]
```

- The `[id]` is the [identifier](https://www.hl7.org/fhir/DSTU2/resource.html#id) of the FHIR Observation resource.
- The `[base]` is the URL of the Spine endpoint [TO BE CONFIRMED].
- Note: The mime-type can be specified to request either XML or JSON using another URL parameter `?_format=[mime-type]`, or a `Content-Type` HTTP header as per the [FHIR specification](https://www.hl7.org/fhir/http.html#mime-type).

### Read Response ###

Success:

- SHALL return a `200` **OK** HTTP status code on successful execution of the interaction.
- SHALL return one `Observation` resource that conforms to the `spine-vm-observation-1` profile.
- SHALL include the `versionId` and `fullUrl` of the current version of the `observation` resource.

```json
{
"resourceType": "Bundle",
  "id": "6f759a10-d1b6-11e6-9598-0800200c9a66",
  "meta": {
    "profile": [
      "http://fhir.nhs.net/StructureDefinition/spine-searchset-bundle-1"
    ]
  },
  "type": "searchset",
  "entry": [
    {
      "resource": {
        "resourceType": "Observation",
        "id": "76e39290-d1aa-11e6-9598-0800200c9a66",
        "meta": {
          "profile": [
            "http://fhir.nhs.net/StructureDefinition/spine-vm-observation-1"
          ]
        },
        "status": "final",
        "code": {
          "coding": [
            {
              "system": "http://fhir.nhs.net/fhir-observation-code-1",
              "code": "0001",
              "display": "Visitors and Migrants status observation"
            }]
        },
        "subject": {
          "reference": "Patient/e88cdcd0-d1aa-11e6-9598-0800200c9a66",
          "display": "Miss Mary Taylor"
        },
        "effectiveDateTime": "2015-01-01T15:00:00+00:00",
        "issued": "2015-02-01T10:00:00+00:00",
        "component": [
          {
            "code": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-vm-observation-component-1",
                  "code": "bcs",
                  "display": "Basic Chargeable Status"
                }
              ]
            },
            "valueCodeableConcept": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-chargeable-status-1",
                  "code": "y",
                  "display": "Chargeable"
                }]
            }
          },
          {
            "code": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-vm-observation-component-1",
                  "code": "ccs",
                  "display": "Category Chargeable Status"
                }]
            },
            "valueCodeableConcept": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-category-status-1",
                  "code": "F",
                  "display": "F"
                }]
            }
          }
        ]
      }
    }
  ]
}
```

Failure: 

- SHALL return a `4xx` or `5xx` HTTP status code  with an `OperationOutcome` resource that conforms to the `spine-operationoutcome-1` profile containing details of the error that occurred.
- The types of error in an `ObservationOutcome` are defined in the [Spine Error or Warning Code ValueSet](/ValueSets/spine-error-or-warning-code-1.xml)

```json
{
"resourceType": "Bundle",
  "id": "4534b230-d1c6-11e6-9598-0800200c9a66",
  "meta": {
    "profile": [
      "http://fhir.nhs.net/StructureDefinition/spine-searchset-bundle-1"
    ]
  },
  "type": "searchset",
  "entry": [
    {
      "resource": {
        "resourceType": "OperationOutcome",
        "id": "4e2c8890-d1c6-11e6-9598-0800200c9a66",
        "meta": {
          "profile": [
            "http://fhir.nhs.net/StructureDefinition/spine-operationoutcome-1"
          ]
        },
        "issue": [
          {
            "severity": "error",
            "code": "not-found",
            "details": {
              "coding": [
                {
                  "system": "http://fhir.nhs.net/spine-error-or-warning-code-1",
                  "code": "VM-0001"
                }]
            },
            "diagnostics": "No Record Found"
          }]
      }
    }
  ]
}
```

### Example Code ###

#### C# ####

```csharp
var client = new FhirClient("http://gpconnect.fhir.nhs.net/fhir/");
client.PreferredFormat = ResourceFormat.Json;
var query = new string[] { "identifier=http://fhir.nhs.net/Id/nhs-number|P002" };
var bundle = client.Search("Patient", query);
FhirSerializer.SerializeResourceToXml(bundle).Dump();
```

#### Java ####

```java
FhirContext ctx = new FhirContext();
IGenericClient client = ctx.newRestfulGenericClient("http://gpconnect.fhir.nhs.net/fhir/");
Bundle bundle = client.search().forResource(Patient.class)
.where(new TokenClientParam("identifier").exactly().systemAndCode("http://fhir.nhs.net/Id/nhs-number", "P002"))
.encodedXml()
.execute();
```

#### cURL ####

{% include embedcurl.html title="Find a patient" command="curl -X GET -H 'Ssp-From: 0001' -H 'Ssp-To: 0002' -H 'Ssp-InteractionID: urn:nhs:names:services:gpconnect:fhir:rest:search:patient' -H 'Cache-Control: no-cache' -H 'Ssp-TraceID: e623b4de-f6bb-be0c-956d-c4ded0d58fc0' 'http://gpconnect.fhir.nhs.net/fhir/Patient?identifier=http://fhir.nhs.net/Id/nhs-number%7CP002'" %}



