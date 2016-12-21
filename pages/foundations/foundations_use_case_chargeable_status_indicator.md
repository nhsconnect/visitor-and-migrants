---
title: Read Visitors & Migrants Chargeable-Status Indicator
keywords: nhsnumber, chargeable-status
tags: [foundations,use_case]
sidebar: foundations_sidebar
permalink: foundations_use_case_chargeable_status_indicator.html
summary: "Use case to read an overseas visitors & migrants chargeable status indicator."
---

## API Use Case ##

This specification describes a single use case.

## Prerequisites ##

To use this API, the requester:

- SHALL have previously traced the patient's NHS Number using PDS or an equivalent service.

## API Usage ##

Resolve (zero or more) `Patient` resources using a business identifier (i.e. NHS Number).

### Request interaction ###

The `[system]` field SHALL be populated with a valid patient identifier system URL (i.e. `http://fhir.nhs.net/Id/nhs-number`).

#### FHIR Relative Request ####

```http
 GET [base]/Observation?subject:Patient.identifier=[system]|[value]
```
```http
 GET /Patient/[id]/Observation
```


#### FHIR Absolute Request ####

```http
GET https://[proxy_server]/https://[spine_server]/[fhir_base]/Observation?subject:Patient.identifier=[system]|[value]
```
```http
GET https://[proxy_server]/https://[spine_server]/[fhir_base]/Patient/[id]/Observation
```

#### Request Headers ####

Consumers SHALL include the following additional HTTP request headers:

| Header               | Value |
|----------------------|-------|
| `Ssp-TraceID`        | Consumer's TraceID (i.e. GUID/UUID) |
| `Ssp-From`           | Consumer's ASID |
| `Ssp-To`             | Provider's ASID |
| `Ssp-InteractionID`  | `urn:nhs:names:services:gpconnect:fhir:rest:search:patient`|

#### Payload Request Body ####

N/A

#### Error Handling ####

Provider systems:

- SHALL return an [OperationOutcome](https://www.hl7.org/fhir/DSTU2/operationoutcome.html) resource that provides additional detail when one or more request fields are corrupt or a specific business rule/constraint is breached.

For example the:

- Business identifier `[system]` is not recognised.
- Business identifier fails structural validation checks (i.e. not enough digits to be a valid NHS Number).

### Request Response ###

#### Response Headers ####

Provider systems are not expected to add any specific headers beyond that described in the HTTP and FHIR&reg; standards.

#### Payload Response Body ####

Spine system:

Success:

- SHALL return a `200` **OK** HTTP status code on successful execution of the interaction.
- SHALL return zero or more matching `Observation` resources in a `Bundle` of `type` searchset.
- SHALL return the `observation` resource that conforms to the `spine-vm-observation-1` profile.
- SHALL include the `versionId` and `fullUrl` of the current version of the `observation` resource.
- Note: , 
- SHALL return an `OperationOutcome` resource in a `Bundle` of `type` searchset if the interaction is a success, however no Overseas Visitors & Migrants record has been found.

Failure: 

- SHALL return a `4xx` or `5xx` HTTP status code  with an `OperationOutcome` resource that conforms to the `spine-operationoutcome-1` profile if the search cannot be executed (not that there is no match).


```json
{
	"resourceType": "Bundle",
	"type": "searchset",
	"entry": [{
		"fullUrl": "http://gpconnect.fhir.nhs.net/fhir/Patient/2/_history/636064088097580046",
		"resource": {
			"resourceType": "Patient",
			"id": "2",
			"meta": {
				"versionId": "636064088097580046",
				"lastUpdated": "2016-08-10T16:52:39.716+01:00",
				"profile": ["http://fhir.nhs.net/StructureDefinition/gpconnect-patient-1"]
			},
			"identifier": [{
				"system": "http://fhir.nhs.net/Id/nhs-number",
				"value": "P002"
			}],
			"name": [{
				"use": "official",
				"family": ["Jackson"],
				"given": ["Jane"],
				"prefix": ["Miss"]
			}],
			"gender": "female",
			"birthDate": "22/02/1982"
		}
	}]
}
```

## Example Code ##

### C# ###

```csharp
var client = new FhirClient("http://gpconnect.fhir.nhs.net/fhir/");
client.PreferredFormat = ResourceFormat.Json;
var query = new string[] { "identifier=http://fhir.nhs.net/Id/nhs-number|P002" };
var bundle = client.Search("Patient", query);
FhirSerializer.SerializeResourceToXml(bundle).Dump();
```

### Java ###

```java
FhirContext ctx = new FhirContext();
IGenericClient client = ctx.newRestfulGenericClient("http://gpconnect.fhir.nhs.net/fhir/");
Bundle bundle = client.search().forResource(Patient.class)
.where(new TokenClientParam("identifier").exactly().systemAndCode("http://fhir.nhs.net/Id/nhs-number", "P002"))
.encodedXml()
.execute();
```

### cURL ###

{% include embedcurl.html title="Find a patient" command="curl -X GET -H 'Ssp-From: 0001' -H 'Ssp-To: 0002' -H 'Ssp-InteractionID: urn:nhs:names:services:gpconnect:fhir:rest:search:patient' -H 'Cache-Control: no-cache' -H 'Ssp-TraceID: e623b4de-f6bb-be0c-956d-c4ded0d58fc0' 'http://gpconnect.fhir.nhs.net/fhir/Patient?identifier=http://fhir.nhs.net/Id/nhs-number%7CP002'" %}