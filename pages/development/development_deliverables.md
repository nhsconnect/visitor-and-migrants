---
title: Developer Cheat Sheet
keywords: development deliverables
tags: [development]
sidebar: overview_sidebar
permalink: development_deliverables.html
summary: "Developer Cheat Sheet shortcuts for the <br/>technical build of Visitors and Migrants API."
---

[TODO: Insert a picture here to show the overall process (e.g. TLS, Setting Audit headers, etc)]


Visitors and Migrants Profiles:

| Profile | Example | ValueSets | Sample Code |
| :--------- |:-----: |:-----: |
| [Spine-VM-Observation-1](StructureDefinitions/Spine-VM-Observation-1.json) | [Observation](Examples/Observation.json) | [fhir-observation-code-1](ValueSets/fhir-observation-code-1.json) <br /> [spine-vm-observation-component-1](ValueSets/spine-vm-observation-component-1.json) <br /> [spine-chargeable-status-1](ValueSets/spine-chargeable-status-1.json) <br /> [spine-category-status-1](ValueSets/spine-category-status-1.json) | [Development Example Code - Coming Soon] |
| [Spine-OperationOutcome-1](StructureDefinitions/Spine-OperationOutcome-1.json) | [OperationOutcome](Examples/OperationOutcome.json) | [spine-error-or-warning-code-1](ValueSets/spine-error-or-warning-code-1.json) | |

Audit Profiles:

| Profile | Example | ValueSets | Sample Code |
| :--------- |:-----: |:-----: |
| [Audit-Patient-1](Audit/StructureDefinitions/Audit-Patient-1.json) | [Patient](Audit/Examples/Patient.json) |  | [Development Example Code - Coming Soon] |
| [Audit-Device-1](Audit/StructureDefinitions/Audit-Device-1.json) | [Device](Audit/Examples/Device.json) | [device-type-codes-snct-1](Audit/ValueSets/device-type-codes-snct-1.json) | |
| [Audit-Organization-1](Audit/StructureDefinitions/Audit-Organization-1.json) | [Organisation](Audit/Examples/Organization.json) | | |
| [Audit-Practitioner-1](Audit/StructureDefinitions/Audit-Practitioner-1.json) | [Practitioner](Audit/Examples/Practitioner.json) | | |

