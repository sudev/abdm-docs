+++
title = "By Demographics"
date = 2023-03-16T09:30:25+05:30
weight = 2
chapter = true
pre = "<b>2.2.2 </b>"
+++

# By Demographics
|  Applicable To                             |   HMIS / LMIS (PVT)  |   Government Health App  |    
|-------------------------------|:----------------------:|:--------------------:|
|   Scan User ABHA QR/Demographics                     |  {{% badge style="blue" %}} Mandatory{{% /badge %}}       |  {{% badge style="blue" %}} Mandatory{{% /badge %}}        |  

## Functionality Overview

ABDM requires HMIS/LMIS to generate a link token that can be used for linking of health records. A demographic authentication where the ABHA address, name, year of birth and gender (as in the ABHA address) is required to generate this linking token.

The user's demographic information can be obtained either via:
1. Scanning the Health Faciilty QR Code
2. By (Health Facility) Scanning the User's ABHA QR Code
3. By OTP

Health Facility can scan the **User's ABHA QR code**, whereby User provides consent to link the Health Record to their account.

Health Facility (HIP) Initiated linking is the process through which an HIP links the patient’s care context (health record) with the patient ABHA Address, after patient registration and creation of health records (in their HMIS/LMIS system)


**Sample QR Codes**

QR Code|JSON output
| -------- | ------- |
Sample QR Code in User's ABHA Application ![ABHA App QR Code](../abha-qr-in-app.png) | ![ABHA App QR Scan](../json-abha-app-qr-scan.png)
Sample QR Code in User's ABHA Card ![ABHA App QR Code](../abha-card-eg.png) | ![ABHA App QR Scan](../json-abha-card-qr.png)

Care context (Health record) linking happens in two steps:

1. Link token generation
2. Linking care context with ABHA address after obtaining valid link token

**Link Token generation**

To achieve linking, the Health Facility (HIP) needs to have a valid link token. There are two possible ways to get the link token:
1. Through scan and share profile use case
2. Using link token service

The link token will be used for linking multiple number of care contexts, and concurrent linkages.

**Pre-requisites:**
1. Patient must have a ABHA app with a valid ABHA address or physical card like a ABHA card
2. Hospital must have a 2D Barcode scanner available at the registration counter.
	- The HRP software registration page have text box to load data scanned from ABHA QR Code.

**Steps to scan User's ABHA QR Code**

1. Hospital operator scans the QR code on patient ABHA card or as displayed in ABHA mobile App
	- The Patient details like ABHA address, Name, DOB, Gender, State Id, District Id, etc are embedded in the QR code as a JSON
2. HRP software must validate the scanned data and collect a Linking Token from ABDM Gateway
3. The software must use the information scanned to register the patient and save the ABHA Address

## Sample User Experience

Here's how the user journey looks on the Health Facility's side:

![Scan User ABHA QR Code](/abdm-docs/img/scan-user-qr.gif)


## Test Cases:

S.no.|Applicable To | Test Summary | Test Scenario |
|--| --| ----------- | ------------------- |
1.|{{% badge %}}Optional{{% /badge %}} Applicable to All (VRFY_ABHA_501)| System must allow scanning of ABHA QR code to read the user's ABHA information|EMR/HMIS scans the user's ABHA QR code

{{% notice title="API Versions" %}}

- **V0.5 API**
  - [Sequence Diagram for V0.5 API](#v05-api-sequence-diagram)
  - [API Information Request Response for V0.5](#v05-api-information-request-response)
- **V3 API**
  - [Sequence Diagram for V3 API](#v3-api-sequence-diagram)
  - [API Information Request Response for V3](#v3-api-information-request-response)

**Note:** V0.5 APIs are currently in production. V3 APIs are currently only available in sandbox
{{% /notice %}}

## V0.5 API Sequence Diagram

{{< mermaid >}}
%%{init:{"fontSize": "1.0rem", "sequence":{"showSequenceNumbers":true}}}%%
sequenceDiagram
title V0.5 APIs - Scan User's ABHA QR Code to verify ABHA Address
actor User
HRP/HIP->>User: Health Facility scans User's ABHA QR code
HRP/HIP->>HIE-CM: POST /v0.5/users/auth/init
HIE-CM-->>HRP/HIP: Response 200
HIE-CM->>HRP/HIP: POST /v0.5/users/auth/on-init
HRP/HIP-->>HIE-CM: Response 200
HRP/HIP->>HIE-CM: /v0.5/users/auth/confirm
HIE-CM-->>HRP/HIP: Response 200
HRP/HIP->>HIE-CM: /v0.5/users/auth/on-confirm
HIE-CM-->>HRP/HIP: Response 200
{{< /mermaid >}}


## V0.5 API Information Request Response

**1. Initiate Authentication of users**

This API is called by HIPs to initiate authentication of users. A transactionId is retuned by the corresponding callback API for confirmation of user auth.

**BASE URLs:**  https://dev.abdm.gov.in/gateway

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/init" >}}

**2. Call Back API For Init**

Response to user authentication initialization from HIP. If the patient's id is valid, CM will return a transactionId as initialization of user auth. If the request is valid, then 'auth.mode' will convey how the authentication should be done.

**BASE URLs:**  https://your-hrp-server.com

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/on-init" >}}

**3. Confirm Authentication Of Users**

This API is called by HIP/HIUs to confirm authentication of users. The transactionId returned by the previous callback API /users/auth/on-init must be sent. If Authentication is successful the callback API will send an "access token" for subsequent purpose specific API calls.

**BASE URLs:**  https://dev.abdm.gov.in/gateway

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/confirm" >}}

**4. Call Back API For Confirm**

This API is called by CM to confirm authentication of users.

**BASE URLs:**  https://your-hrp-server.com

{{< swaggermin src="/abdm-docs/Yaml/ndhm-hip.yml" api="POST /v0.5/users/auth/on-confirm" >}}


## V3 API Sequence Diagram

{{< mermaid >}}
%%{init:{"fontSize": "1.0rem", "sequence":{"showSequenceNumbers":true}}}%%
sequenceDiagram
title V3 API- Scan User's ABHA QR Code to verify ABHA Address
actor User
HRP/HIP->>User:Health Facility scans User's ABHA QR code
HRP/HIP->>ABDM Sandbox Gateway:POST /v3/token/generate-token
ABDM Sandbox Gateway-->>HRP/HIP:Response 200
ABDM Sandbox Gateway->>HRP/HIP:POST /v3/token/on-generate-token
HRP/HIP-->>ABDM Sandbox Gateway:Response 200
{{< /mermaid >}}

## V3 API Information Request Response

**1. Generate link token**

**BASE URLs:**
  - Sandbox Server URL: https://dev.abdm.gov.in/hiecm/api/

  - Production Server URL: https://live.abdm.gov.in/hiecm/api

{{< swaggermin src="/abdm-docs/Yaml/HIECM-LinkTokenService.yml" api="POST /v3/token/generate-token" >}}

**2. Call back API**

**BASE URLs:**
  - Sandbox Server URL: https://dev.abdm.gov.in/hiecm/api/

  - Production Server URL: https://live.abdm.gov.in/hiecm/api

{{< swaggermin src="/abdm-docs/Yaml/HIECM-LinkTokenService.yml" api="POST /{callback_url}/v3/hip/token/on-generate-token$" >}}

{{%icon icon="info-circle" %}} If you are unauthorised for V3 APIS, kindly contact Integration.support@nha.gov.in with your clientID.


## Verify ABHA Address by demographic

If Linking token has expired (more than 6 months old or it has been already used once). In that case, all the demographic information is available and can be used directly.