---
title: Introduction to Visitors and Migrants API
keywords: homepage
tags: [overview]
sidebar: overview_sidebar
permalink: index.html
toc: false
summary: A brief introduction to getting started with the FHIR&reg; APIs.
---

{% include important.html content="This site is under active development by NHS Digital and is intended to provide all the technical resources you need to successfully develop the APIs. This project is being developed using an agile methodology so iterative updates to content will be added on a regular basis." %}

{% include warning.html content="This site is provided for information only and is intended for those engaged with NHS Digital on the development of the APIs. It is advised not to develop against these specifications until a formal announcement has been made." %}

# Introduction #
Visitors and Migrants aims to support the Department of Health (DoH) Cost Recovery program for secondary care treatment provided to patients that are not eligible to receive free NHS care.

In 2013 a public consultation on the existing charging arrangements for overseas visitors and migrants was launched by the DoH. As a result of this consultation changes have been introduced to determine how overseas visitors and migrants are identified and charged for their healthcare.

Furthermore, an Immigration Health Surcharge (IHS) was introduced on 6 April 2015 under the provisions of the Immigration Act 2014 and the Immigration (Health Charge) Order 2015. From 6 April 2015, nationals of countries outside the European Economic Area (EEA) 4 who apply for leave to enter the UK for more than six months, or who apply to extend their stay, in most immigration categories will pay the ‘Immigration Health Surcharge’, with the income going to the NHS.

An intrinsic requirement to support the measures detailed above is to provide an indication on Spine of whether or not an NHS patient may be chargeable for NHS care. In support of this, NHS Digital has implemented the functionality to provide this indication via the Summary Care Record Application (SCRa) for a limited set of patients, i.e. those patients who have been identified as having paid the IHS; are exempt from paying or may be chargeable for NHS care.

# Development Phases #
The DoH Cost Recovery programme consists of five development phases that are outlined as part of the high-level maturity model. This site will primarily focus on Phase 3.

# Using this guide #

This guide has been created to support the adoption of profiles and FHIR resources. As such the site is structured around stakeholders including API users, developers and architects.  

{% include custom/api_overview.svg %}

The above steps outline a complete API journey from imagination and exploring to developing local APIs using profiles all the way to deploying a live API.

{% include custom/contribute.html content="If you want to get involved in any part of this then please get in touch with careconnect@interopen.org "%}

# Focus #

The current site focuses on a typical API Developer's Journey as highlighted by the green boxes below in the developer journey:

<img src="images/roadmap/guide-focus.png" style="width:100%;max-width: 100%;">

NHS Digital is contributing to progressing the profile developmenet.

Please see the explanation of the complete development roadmap.

{% include custom/contribute.html content="Please contact [careconnect@interopen.org] to get involved." %}



# Resource Roadmap #

The [API journey](overview_api_journey.html) outlines the development roadmap for the RESTful APIs outlined within this site.

<img src="images/roadmap/roadmap-online.png" style="width:100%;max-width: 100%;">

The above roadmap illustrates the steps necessary to create, test and verify the profiles as well as some of the supporting tooling which might be necessary to build to provide viable APIs.

{% include custom/contribute.html content="To get involved in any parts of the roadmap or to discuss the other elements please get in touch with careconnect@interopen.org "%}
