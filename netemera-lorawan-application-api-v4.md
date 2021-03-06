# Netemera LoRaWAN Application API V4

## Table of Contents

- [Introduction](#introduction)
- [Authorization](#authorization)
  - [Register Client](#register-client)
  - [Obtain Access Token With Client Credentials](#obtain-access-token-with-client-credentials)
  - [Obtain Authorization Code](#obtain-authorization-code)
  - [Obtain Access Token With Authorization Code](#obtain-access-token-with-authorization-code)
  - [Obtain Access Token With Refresh Token](#obtain-access-token-with-refresh-token)
- [Applications](#applications)
  - [Application](#application)
  - [Retrieve Applications](#retrieve-applications)
  - [Retrieve Application](#retrieve-application)
  - [Create Application](#create-application)
  - [Update Application](#update-application)
  - [Delete Application](#delete-application)
- [Application Integrations](#application-integrations)
  - [Akenza Core](#akenza-core)
    - [Akenza Core Application Integration](#akenza-core-application-integration)
    - [Retrieve Akenza Core Application Integration](#retrieve-akenza-core-application-integration)
    - [Create Akenza Core Application Integration](#create-akenza-core-application-integration)
    - [Update Akenza Core Application Integration](#update-akenza-core-application-integration)
    - [Delete Akenza Core Application Integration](#delete-akenza-core-integration)
- [Device Profiles](#device-profiles)
  - [Device Profile](#device-profile)
  - [Retrieve Device Profiles](#retrieve-device-profiles)
  - [Retrieve Device Profile](#retrieve-device-profile)
  - [Create Device Profile](#create-device-profile)
  - [Update Device Profile](#update-device-profile)
  - [Delete Device Profile](#delete-device-profile)
- [End-Devices](#end-devices)
  - [End-Device](#end-device)
  - [Retrieve Application End-Devices](#retrieve-application-end-devices)
  - [Retrieve End-Device](#retrieve-end-device)
  - [Create Application End-Device](#create-application-end-device)
  - [Update End-Device](#update-end-device)
  - [Delete End-Device](#delete-end-device)
- [End-Device Activations](#end-device-activations)
  - [End-Device Activation](#end-device-activation)
  - [Retrieve End-Device Activation](#retrieve-end-device-activation)
  - [Create End-Device Activation](#create-end-device-activation)
  - [Update End-Device Activation](#update-end-device-activation)
  - [Delete End-Device Activation](#delete-end-device-activation)
- [Uplink Packets](#uplink-packets)
  - [Uplink Packet](#uplink-packet)
    - [Gateway Information](#gateway-information)
  - [Receive End-Device Uplink Packets](#receive-end-device-uplink-packets)
  - [Receive Application Uplink Packets](#receive-application-uplink-packets)
- [Downlink Packets](#downlink-packets)
  - [Downlink Packet](#downlink-packet)
  - [Send End-Device Downlink Packet](#send-end-device-downlink-packet)
  - [Receive End-Device Downlink Packets](#receive-end-device-downlink-packets)

## Introduction

Netemera LoRaWAN Application API is a HTTP-based interface enabling programmatic management of and communication with end-devices activated in the network.

The interface allows for the management of:

* Applications, used to group end-devices into logical collections, typically communicating with a single user application.
* Device profiles that include information about end-device capabilities and boot parameters that are needed for setting up the LoRaWAN™ radio access service.
* The actual end-devices, their attributes and activation status in the network.

A client can subscribe to and receive live and historical data (uplink packets) sent by activated end-devices. A separate endpoint allows for sending data (downlink packets) to end-devices.

The API is available at the following base URL (EU region): [https://application.lorawan.netemera.com/api/v4](https://application.lorawan.netemera.com/api/v4) and combines RESTful and [SSE (Server-Sent Events)](https://www.w3.org/TR/eventsource/) endpoints. Requests and response bodies are formatted in [JSON](https://www.json.org/) and the [OAuth 2.0](https://tools.ietf.org/html/rfc6749) protocol is used for authorization.

> **IMPORTANT:** You cannot run any of the sample requests in this guide as-is. Replace call-specific parameters such as host names, tokens, IDs, and secrets with your own values.

## Authorization

All API calls must include an OAuth 2.0 access token, either as the `Authorization` HTTP header (recommended):

`Authorization: Bearer ACCESS_TOKEN`

or as the `access_token` URL query parameter:

`access_token=ACCESS_TOKEN`.

### Register Client

Before starting with the OAuth 2.0 authorization process, a client (a third-party app) must be registered with [Netemera Authorization Server](https://authorization.netemera.com). Each client will be provided with a client ID and a client secret. The client secret must be kept confidential.

If the client wants to make calls on behalf of a resource owner (user), a redirect URI must also be registered. The redirect URI will be used to pass authorization codes to the app allowing it to obtain access tokens for accessing resource owner's resources.

### Obtain Access Token with Client Credentials

This method is applicable to clients wanting to access resources on their own behalf.

#### Endpoint

POST https://authorization.netemera.com/oauth2/token

#### Request Parameters

The parameters must be added using the "application/x-www-form-urlencoded" format as described in [Appendix B of RFC6749](https://tools.ietf.org/html/rfc6749#appendix-B) to the request entity-body.

Parameter|Type|Required|Description
---|---|---|---
`grant_type`|string|true| Must equal "client_credentials"
`audience`|string|true|The target API's URL the access token will be generated for. Must equal "https://application.lorawan.netemera.com/api/v4"

#### Request Headers

Header|Required|Description
---|---|---
`Authorization: Basic {token}`|true|The `token` parameter is a base64-encoding of the following string "CLIENT_ID:CLIENT_SECRET" where client ID and secret are user name and password given to the client

#### Response Body

An object with the following attributes:

Field|Type|Optional|Description
---|---|---|---
`access_token`|string|false|An access token
`token_type`|string|false|The type of the token
`expires_in`|string|false|The validity time of the token in seconds
`refresh_token`|string|false|A refresh token

#### Sample Request

```shell
curl \
  --request POST \
  --url 'https://authorization.netemera.com/oauth2/token' \
  --user 'CLIENT_ID:CLIENT_SECRET' \
  --data 'grant_type=client_credentials&audience=https://application.lorawan.netemera.com/api/v4'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token": "ACCESS_TOKEN",
  "token_type": "Bearer",
  "expires_in": 86400,
  "refresh_token": "REFRESH_TOKEN"
}
```

### Obtain Authorization Code

To obtain an authorization code for accessing resources on behalf of a resource owner, a client must orchestrate an approval interaction between the resource owner and the authorization server. This is done by redirecting the resource owner's user-agent to an authorization endpoint and later receiving the authorization code via a call to the endpoint identified by the redirect URI.

#### Endpoint

GET https://authorization.netemera.com/oauth2/authorize

#### Request Parameters

The parameters must be added to the query part of the URI.

Parameter|Type|Required|Description
---|---|---|---
`response_type`|string|true| Must equal "code"
`client_id`|string|true| The client ID
`redirect_uri`|string|false| The redirect URI, as defined during the [client registration procedure](register-client)
`scope`|string|false| The requested scope
`state`|string|false| The state value
`audience`|string|true| The target API's URL the access token will be generated for. Must equal "https://application.lorawan.netemera.com/api/v4"

#### Request Headers

Header|Required|Description
---|---|---
`Authorization: Basic {token}`|true|The `token` parameter is a base64-encoding of the following string "RESOURCE_OWNER_ID:RESOURCE_OWNER_SECRET" where RESOURCE_OWNER_ID and RESOURCE_OWNER_SECRET are user name and password given to the user in the registration process.

#### Response Body

An object with the following attributes:

Field|Type|Optional|Description
---|---|---|---
`access_token`|string|false|An access token
`token_type`|string|false|The type of the token
`expires_in`|string|false|The validity time of the token in seconds
`refresh_token`|string|false|A refresh token

#### Sample Request

```shell
curl \
  --request GET \
  --url 'https://authorization.netemera.com/oauth2/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=SCOPE&state=STATE&audience=https://application.lorawan.netemera.com/api/v4' \
  --user 'RESOURCE_OWNER_ID:RESOURCE_OWNER_SECRET'
```

#### Sample Response

```http
HTTP/1.1 302 Found
Location: REDIRECT_URI?code=AUTHORIZATION_CODE
```

### Obtain Access Token with Authorization Code

This method is applicable to apps wanting to access resources on behalf of other resource owners. Obtaining an authorization code is described in [Obtain Authorization Code](#obtain-authorization-code) section.

#### Endpoint

POST https://authorization.netemera.com/oauth2/token

#### Request Parameters

The parameters must be added using the "application/x-www-form-urlencoded" format as described in [Appendix B of RFC6749](https://tools.ietf.org/html/rfc6749#appendix-B) in the request entity-body.

Parameter|Type|Required|Description
---|---|---|---
`grant_type`|string|true| Must equal "authorization_code"
`code`|string|true| The authorization code

#### Request Headers

Header|Required|Description
---|---|---
`Authorization: Basic {token}`|true|The `token` parameter is a base64-encoding of the following string "CLIENT_ID:CLIENT_SECRET" where CLIENT_ID and CLIENT_SECRET are the ID and password given to the client

#### Response Body

An object with the following attributes:

Field|Type|Optional|Description
---|---|---|---
`access_token`|string|false|An access token
`token_type`|string|false|The type of the token
`expires_in`|string|false|The validity time of the token in seconds
`refresh_token`|string|false|A refresh token

#### Sample Request

```shell
curl \
  --request POST \
  --url 'https://authorization.netemera.com/oauth2/token' \
  --user 'CLIENT_ID:CLIENT_SECRET' \
  --data 'grant_type=authorization_code&code=AUTHORIZATION_CODE'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token": "ACCESS_TOKEN",
  "token_type": "Bearer",
  "expires_in": 86400,
  "refresh_token": "REFRESH_TOKEN"
}
```

### Obtain Access Token with Refresh Token

#### Endpoint

POST https://authorization.netemera.com/oauth2/token

#### Request Parameters

The parameters must be added using the "application/x-www-form-urlencoded" format as described in [Appendix B of RFC6749](https://tools.ietf.org/html/rfc6749#appendix-B) in the request entity-body.

Parameter|Type|Required|Description
---|---|---|---
`grant_type`|string|true|Must equal "refresh_token"
`refresh_token`|string|true|The refresh token

#### Request Headers

Header|Required|Description
---|---|---
`Authorization: Basic {token}`|true|The `token` parameter is a base64-encoding of the following string "CLIENT_ID:CLIENT_SECRET" where client ID and secret are user name and password given to the client

#### Response Body

An object with the following attributes:

Field|Type|Optional|Description
---|---|---|---
`access_token`|string|false|An access token
`token_type`|string|false|The type of the token
`expires_in`|string|false|The validity time of the token in seconds
`refresh_token`|string|false|A refresh token

#### Sample Request

```shell
curl \
  --request POST \
  --url 'https://authorization.netemera.com/oauth2/token' \
  --user 'CLIENT_ID:CLIENT_SECRET' \
  --data 'grant_type=refresh_token&refresh_token=REFRESH_TOKEN'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token": "ACCESS_TOKEN",
  "token_type": "Bearer",
  "expires_in": 86400,
  "refresh_token": "REFRESH_TOKEN"
}
```

## Applications

Applications group end-devices into logical collections, typically communicating with a single user application.

### Application

An application has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`applicationId`|string|true|The unique identifier
`name`|string|true|A friendly name

### Retrieve Applications

Retrieves all applications.

#### Request Definition

GET /api/v4/applications

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an array of [Applications](#application)|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request GET \
  --url 'https://application.lorawan.netemera.com/api/v4/applications' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [{
    "type": "application",
    "id": "APPLICATION_ID",
    "attributes": {
      "applicationId": "APPLICATION_ID",
      "name": "My application"
     }
  }]
}
```

### Retrieve Application

Retrieves the application identified by the given application ID.

#### Request Definition

GET /api/v4/applications/{application_id}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`application_id`|string|true|The unique identifier of the application

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Application](#application)|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request GET \
  --url 'https://application.lorawan.netemera.com/api/v4/applications/APPLICATION_ID' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": {
    "type": "application",
    "id": "APPLICATION_ID",
    "attributes": {
      "applicationId": "APPLICATION_ID",
      "name": "My application"
     }
  }
}
```

### Create Application

Creates a new application.

#### Request Definition

POST /api/v4/applications

#### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|

#### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Application](#application).

#### Response

Status|Body|Description
---|---|---
`201 Created`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Application](#application)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request POST \
  --url 'https://application.lorawan.netemera.com/api/v4/applications' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data '{"data":{"type":"application","attributes":{"name":"My application"}}}'
```

#### Sample Response

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "data": {
    "type": "application",
    "id": "APPLICATION_ID",
    "attributes": {
      "applicationId": "APPLICATION_ID",
      "name": "My application"
     }
  }
}
```

### Update Application

Updates the application identified by the given application ID.

#### Request Definition

PATCH /api/v4/applications/{application_id}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`application_id`|string|true|The unique identifier of the application

#### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|

#### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Application](#application).

#### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Application](#application)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request PATCH \
  --url 'https://application.lorawan.netemera.com/api/v4/applications/APPLICATION_ID' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data '{"data":{"type":"application","id":"APPLICATION_ID","attributes":{"applicationId":"APPLICATION_ID","name":"My application"}}}'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{}
```

### Delete Application

Deletes the application identified by the given application ID.

#### Request Definition

DELETE /api/v4/applications/{application_id}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`application_id`|string|true|The unique identifier of the application

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`204 No Content`||Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`404 Not Found`||The application does not exist
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request DELETE \
  --url 'https://application.lorawan.netemera.com/api/v4/applications/APPLICATION_ID' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 204 No Content
```

## Application Integrations

### Akenza Core

#### Akenza Core Application Integration

An Akenza Core application integration has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`applicationId`|string|false|The unique identifier of the application
`gatewayUrl`|string|false|The Akzenza Core gateway URL

#### Retrieve Akenza Core Application Integration

Retrieves the Akenza Core application integration identified by the given application ID.

##### Request Definition

GET /api/v4/applications/{application_id}/integrations/akenza-core

##### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`application_id`|string|true|The unique identifier of the application

##### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

##### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Akenza Core Application Integration](#akenza-core-application-integration)|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

##### Sample Request

```shell
curl \
  --request GET \
  --url 'https://application.lorawan.netemera.com/api/v4/applications/APPLICATION_ID/integrations/akenza-core' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

##### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": {
    "type": "akenza-core-application-integration",
    "id": "APPLICATION_ID",
    "attributes": {
      "applicationId": "APPLICATION_ID",
      "gatewayUrl": "GATEWAY_URL"
     }
  }
}
```

#### Create Akenza Core Application Integration

Creates an Akenza Core application integration.

##### Request Definition

POST /api/v4/applications/{application_id}/integrations/akenza-core

##### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|

##### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Akenza Core Application Integration](#akenza-core-application-integration).

##### Response

Status|Body|Description
---|---|---
`201 Created`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Akenza Core Application Integration](#akenza-core-application-integration)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

##### Sample Request

```shell
curl \
  --request POST \
  --url 'https://application.lorawan.netemera.com/api/v4/applications/APPLICATION_ID/integrations/akenza-core' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data '{"data":{"type":"akenza-core-application-integration","id":"APPLICATION_ID","attributes":{"applicationId":"APPLICATION_ID","gatewayUrl":"GATEWAY_URL"}}}'
```

##### Sample Response

```http
HTTP/1.1 201 Created
Content-Type: application/json

{}
```

#### Update Akenza Core Application Integration

Updates the Akenza Core application integration identified by the given application ID.

##### Request Definition

PATCH /api/v4/applications/{application_id}/integrations/akenza-core

##### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`application_id`|string|true|The unique identifier of the application

##### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|

##### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Akenza Core Application Integration](#akenza-core-application-integration).

##### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Akenza Core Application Integration](#akenza-core-application-integration)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

##### Sample Request

```shell
curl \
  --request PATCH \
  --url 'https://application.lorawan.netemera.com/api/v4/applications/APPLICATION_ID/integrations/akenza-core' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data '{"data":{"type":"akenza-core-application-integration","id":"APPLICATION_ID","attributes":{"applicationId":"APPLICATION_ID","gatewayUrl":"GATEWAY_URL"}}}'
```

##### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{}
```

#### Delete Akenza Core Application Integration

Deletes the Akenza Core application integration identified by the given application ID.

##### Request Definition

DELETE /api/v4/applications/{application_id}/integrations/akenza-core

##### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`application_id`|string|true|The unique identifier of the application

##### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

##### Response

Status|Body|Description
---|---|---
`204 No Content`||Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`404 Not Found`||The application does not exist
`405 Forbidden`||Missing permissions

##### Sample Request

```shell
curl \
  --request DELETE \
  --url 'https://application.lorawan.netemera.com/api/v4/applications/APPLICATION_ID/integrations/akenza-core' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

##### Sample Response

```http
HTTP/1.1 204 No Content
```

## Device Profiles

Device profiles include information about end-device capabilities and boot parameters that are needed for setting up the LoRaWAN radio access service. These information shall be provided by the end-device manufacturer. Every commissioned end-device has to have a device profile assigned.

### Device Profile

A device profile has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`deviceProfileId`|string|true|The unique identifier
`name`|string|true|A friendly name
`supportsClassB`|boolean|false|`true` if an end-device supports class B
`classBTimeout`|integer|true|If class B is supported, the maximum delay, in seconds, for an end-device to answer a MAC request or a confirmed downlink frame
`pingSlotPeriod`|integer|true|If class B is supported, defines for how long, in seconds, an end-device opens a ping slot window.
`pingSlotFrequency`|number|true|If class B is supported, defines how often, in seconds, an end-device opens a ping slot window.
`supportsClassC`|boolean|false|`true` if an end-device supports class C
`classCTimeout`|integer|true|If class C is supported, the maximum delay, in seconds, for an end-device to answer a MAC request or a confirmed downlink frame
`macVersion`|string|false|The version of the LoRaWAN supported by an end-device e.g., 1.0.2
`regParamsRevision`|string|false|The revision of the LoRaWAN Regional parameters supported by an end-device e.g., B
`supportsJoin`|boolean|false|`true` if an end-device supports join (OTAA) or not (ABP)
`rxDelay1`|integer|true|If end-device uses ABP, the class A RX1 delay
`rxDrOffset1`|integer|true|If an end-device uses ABP, RX1 data rate offset
`rxDataRate2`|integer|true|If an end-device uses ABP, RX2 data rate
`rxFreq2`|number|true|If an end-device uses ABP, RX2 channel frequency e.g., 868.1
`factoryPresetFreqs`|array[number]|false|If an end-device uses ABP, a list of factory-preset frequencies e.g., [868.1, 868.3, 868.5]
`maxEirp`|integer|false|The maximum EIRP supported by an end-device
`maxDutyCycle`|number|true|The maximum duty cycle supported by an end-device e.g., 0.01 (indicates 1%)
`rfRegion`|string|false|The RF region name e.g., EU868
`supports32BitFCnt`|boolean|true|`true` if an end-device supports 32-bit frame counter (mandatory for LoRaWAN 1.0 End-Devices)

### Retrieve Device Profiles

Retrieves all device profiles.

#### Request Definition

GET /api/v4/device-profiles

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an array of [Device Profiles](#device-profile)|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request GET \
  --url 'https://application.lorawan.netemera.com/api/v4/device-profiles' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [{
    "type": "device-profile",
    "id": "DEVICE_PROFILE_ID",
    "attributes": {
      "deviceProfileId": "DEVICE_PROFILE_ID",
      "name": "My class-A OTAA device profile",
      "supportsClassB": false,
      "classBTimeout": 1,
      "pingSlotPeriod": 1,
      "pingSlotFrequency": 868.60,
      "supportsClassC": false,
      "classCTimeout": 1,
      "macVersion": "1.0.2",
      "regParamsRevision": "B",
      "supportsJoin": true,
      "rxDelay1": 1,
      "rxDrOffset1": 0,
      "rxDataRate2": 0,
      "rxFreq2": 869.525,
      "factoryPresetFreqs": [868.1, 868.3, 868.5],
      "maxEirp": 14,
      "maxDutyCycle": 0.01,
      "rfRegion": "EU868",
      "supports32BitFCnt": false
     }
  }]
}
```

### Retrieve Device Profile

Retrieves the device profile identified by the given device profile ID.

#### Request Definition

GET /api/v4/device-profiles/{device_profile_id}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`device_profile_id`|string|true|The unique identifier of the device profile

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing a [Device Profile](#device-profile)|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request GET \
  --url 'https://application.lorawan.netemera.com/api/v4/device-profiles/DEVICE_PROFILE_ID' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": {
    "type": "device-profile",
    "id": "DEVICE_PROFILE_ID",
    "attributes": {
      "deviceProfileId": "DEVICE_PROFILE_ID",
      "name": "My class-A OTAA device profile",
      "supportsClassB": false,
      "classBTimeout": 1,
      "pingSlotPeriod": 1,
      "pingSlotFrequency": 868.60,
      "supportsClassC": false,
      "classCTimeout": 1,
      "macVersion": "1.0.2",
      "regParamsRevision": "B",
      "supportsJoin": true,
      "rxDelay1": 1,
      "rxDrOffset1": 0,
      "rxDataRate2": 0,
      "rxFreq2": 869.525,
      "factoryPresetFreqs": [868.1, 868.3, 868.5],
      "maxEirp": 14,
      "maxDutyCycle": 0.01,
      "rfRegion": "EU868",
      "supports32BitFCnt": false
     }
  }
}
```

### Create Device Profile

Creates a device profile.

#### Request Definition

POST /api/v4/device-profiles

#### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|

#### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing a [Device Profile](#device-profile).

#### Response

Status|Body|Description
---|---|---
`201 Created`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing a [Device Profile](#device-profile)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request POST \
  --url 'https://application.lorawan.netemera.com/api/v4/device-profiles' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data '{"data":{"type":"device-profile","attributes":{"name":"My device profile","supportsClassB":false,"classBTimeout":1,"pingSlotPeriod":1,"pingSlotFrequency":868.60,"supportsClassC":false,"classCTimeout":1,"macVersion":"1.0.2","regParamsRevision":"B","supportsJoin":true,"rxDelay1":1,"rxDrOffset1":0,"rxDataRate2":0,"rxFreq2":869.525,"factoryPresetFreqs":[868.1, 868.3, 868.5],"maxEirp":14,"maxDutyCycle":0.01,"rfRegion":"EU868","supports32BitFCnt":false}}}'
```

#### Sample Response

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "data": {
    "type": "device-profile",
    "id": "DEVICE_PROFILE_ID",
    "attributes": {
      "deviceProfileId": "DEVICE_PROFILE_ID",
      "name": "My class-A OTAA device profile",
      "supportsClassB": false,
      "classBTimeout": 1,
      "pingSlotPeriod": 1,
      "pingSlotFrequency": 868.60,
      "supportsClassC": false,
      "classCTimeout": 1,
      "macVersion": "1.0.2",
      "regParamsRevision": "B",
      "supportsJoin": true,
      "rxDelay1": 1,
      "rxDrOffset1": 0,
      "rxDataRate2": 0,
      "rxFreq2": 869.525,
      "factoryPresetFreqs": [868.1, 868.3, 868.5],
      "maxEirp": 14,
      "maxDutyCycle": 0.01,
      "rfRegion": "EU868",
      "supports32BitFCnt": false
     }
  }
}
```

### Update Device Profile

Updates the device profile identified by the given device profile ID.

#### Request Definition

PATCH /api/v4/device-profiles/{device_profile_id}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`device_profile_id`|string|true|The unique identifier of the device profile

#### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|

#### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing a [Device Profile](#device-profile).

#### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing a [Device Profile](#device-profile)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request PATCH \
  --url 'https://application.lorawan.netemera.com/api/v4/device-profiles/DEVICE_PROFILE_ID' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data '{"data":{"type":"device-profile","id":"DEVICE_PROFILE_ID","attributes":{"deviceProfileId":"DEVICE_PROFILE_ID","name":"My device profile","supportsClassB":false,"classBTimeout":1,"pingSlotPeriod":1,"pingSlotFrequency":868.60,"supportsClassC":false,"classCTimeout":1,"macVersion":"1.0.2","regParamsRevision":"B","supportsJoin":true,"rxDelay1":1,"rxDrOffset1":0,"rxDataRate2":0,"rxFreq2":869.525,"factoryPresetFreqs":[868.1, 868.3, 868.5],"maxEirp":14,"maxDutyCycle":0.01,"rfRegion":"EU868","supports32BitFCnt":false}}}'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{}
```

### Delete Device Profile

Deletes the device profile identified by the given device profile ID.

#### Request Definition

DELETE /api/v4/device-profiles/{device_profile_id}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`device_profile_id`|string|true|The unique identifier of the device profile

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`204 No Content`||Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`404 Not Found`||The device profile does not exist
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request DELETE \
  --url 'https://application.lorawan.netemera.com/api/v4/device-profiles/DEVICE_PROFILE_ID' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 204 No Content
```

## End-Devices

### End-Device

An end-device has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`devEui`|string|false|The EUI-64
`name`|string|true|A friendly name
`applicationId`|string|false|The unique identifier of the [Application](#application)
`deviceProfileId`|string|false|The unique identifier of the [Device Profile](#device-profile)
`nwkKey`|string|true|The network key
`appKey`|string|true|The application key

### Retrieve Application End-Devices

Retrieves application end-devices.

#### Request Definition

GET /api/v4/applications/{application_id}/end-devices

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`application_id`|string|true|The unique identifier of the application

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an array of [End-Devices](#end-device)|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request GET \
  --url 'https://application.lorawan.netemera.com/api/v4/applications/APPLICATION_ID/end-devices' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [{
    "type": "end-device",
    "id": "DEV_EUI",
    "attributes": {
      "devEui": "DEV_EUI",
      "name": "My end-device",
      "applicationId": "APPLICATION_ID",
      "deviceProfileId": "DEVICE_PROFILE_ID",
      "nwkKey": "000000000000000000000000000000000000",
      "appKey": "000000000000000000000000000000000000",
     }
  }]
}
```

### Retrieve End-Device

Retrieves the end-device identified by the given EUI-64.

#### Request Definition

GET /api/v4/end-devices/{dev_eui}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [End-Device](#end-device)|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request GET \
  --url 'https://application.lorawan.netemera.com/api/v4/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": {
    "type": "end-device",
    "id": "DEV_EUI",
    "attributes": {
      "devEui": "DEV_EUI",
      "name": "My end-device",
      "applicationId": "APPLICATION_ID",
      "deviceProfileId": "DEVICE_PROFILE_ID",
      "nwkKey": "000000000000000000000000000000000000",
      "appKey": "000000000000000000000000000000000000",
     }
  }
}
```

### Create Application End-Device

Creates a new end-device in the given application.

#### Request Definition

POST /api/v4/applications/{application_id}/end-devices

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`application_id`|string|true|The unique identifier of the application

#### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|


#### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [End-Device](#end-device).

#### Response

Status|Body|Description
---|---|---
`201 Created`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [End-Device](#end-device)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request POST \
  --url 'https://application.lorawan.netemera.com/api/v4/applications/APPLICATION_ID/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data '{"data":{"type":"end-device","id":"DEV_EUI","attributes":{"devEui":"DEV_EUI","name":"My end-device","applicationId":"APPLICATION_ID","deviceProfileId": "DEVICE_PROFILE_ID","nwkKey":"000000000000000000000000000000000000","appKey":"000000000000000000000000000000000000"}}}'
```

#### Sample Response

```http
HTTP/1.1 201 Created
```

### Update End-Device

Updates the end-device identified by the given EUI.

#### Request Definition

PATCH /api/v4/end-devices/{dev_eui}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device

#### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|

#### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [End-Device](#end-device).

#### Response

Status|Body|Description
---|---|---
`201 Created`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an  [End-Device](#end-device)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request PATCH \
  --url 'https://application.lorawan.netemera.com/api/v4/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data '{"data":{"type":"end-device","id":"DEV_EUI","attributes":{"devEui":"DEV_EUI","name":"test","applicationId":"APPLICATION_ID","deviceProfileId": "DEVICE_PROFILE_ID","nwkKey":"000000000000000000000000000000000000","appKey":"000000000000000000000000000000000000",}}}'
```

#### Sample Response

```http
HTTP/1.1 201 Created
```

### Delete End-Device

Deletes the end-device identified by the given EUI.

#### Request Definition

DELETE /api/v4/end-devices/{dev_eui}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`204 No Content`||Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`404 Not Found`||The application does not exist
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request DELETE \
  --url 'https://application.lorawan.netemera.com/api/v4/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 204 No Content
```

## End-Device Activations

To participate in a LoRaWAN network, each end-device has to be activated. Activation of an end-device can be achieved in two ways, either automatically via Over-The-Air Activation (OTAA) when an end-device is deployed or reset, or via Activation By Personalization (ABP) in which an activation record has to be created manually.

### End-Device Activation

An end-device activation has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`devEui`|string|false|The EUI-64 of the end-device
`devAddr`|string|true|The network address of the end-device
`fNwkSIntKey`|string|false|The network session key specific for the end-device. It is used by the end-device to calculate the MIC or part of the MIC (message integrity code) of all uplink data messages. For LoRaWAN 1.0.x devices this is the equivalent of NwkSKey.
`sNwkSIntKey`|string|false|The network session key specific for the end-device. It is used by the end-device to verify the MIC (message integrity code) of all downlink data messages to ensure data integrity and to compute half of the uplink messages MIC. For LoRaWAN 1.0.x devices this is the equivalent of NwkSKey.
`nwkSEncKey`|string|false|The network session key specific to the end-device. It is used to encrypt & decrypt uplink & downlink MAC commands transmitted as payload on port 0 or in the FOpt field. For LoRaWAN 1.0.x devices this is the equivalent of NwkSKey.
`appSKey`|string|false|The application session key specific for the end-device. It is used by both the application server and the end-device to encrypt and decrypt the payload field of application-specific data messages.
`fCntUp`|integer|false|The number of data frames sent uplink by the end-device to the network server
`nFCntDown`|integer|false|The number of MAC communication data frames sent by network server to the end-device. For LoRaWAN 1.0.x devices this is the equivalent of FCntDown.
`aFCntDown`|integer|false|The number of data frames sent downlink to the end-device by the application server. For LoRaWAN 1.0.x devices this is the equivalent of FCntDown.

### Retrieve End-Device Activation

Retrieves the activation for the given end-device.

#### Request Definition

GET /api/v4/end-devices/{dev_eui}/activation

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|s

#### Response

Status|Body|Description
---|---|---
`200 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [End-Device Activation](#end-device-activation)|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request GET \
  --url 'https://application.lorawan.netemera.com/api/v4/end-devices/DEV_EUI/activation' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": {
    "type": "end-device-activation",
    "id": "DEV_EUI",
    "attributes": {
      "devEui": "DEV_EUI",
      "devAddr": "00000000",
      "fNwkSIntKey": "00000000000000000000000000000000",
      "sNwkSIntKey": "00000000000000000000000000000000",
      "nwkSEncKey": "00000000000000000000000000000000",
      "appSKey": "00000000000000000000000000000000",
      "fCntUp": 0,
      "nFCntDown": 0,
      "aFCntDown": 0
     }
  }
}
```

### Create End-Device Activation

Creates a new activation for the given end-device.

#### Request Definition

POST /api/v4/end-devices/{dev_eui}/activation

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device

#### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|

#### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [End-Device Activation](#end-device-activation).

#### Response

Status|Body|Description
---|---|---
`201 Created`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [End-Device Activation](#end-device-activation)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request POST \
  --url 'https://application.lorawan.netemera.com/api/v4/end-devices/DEV_EUI/activation' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{"data":{"type":"end-device-activation","id":"DEV_EUI","attributes":{"devEui":"DEV_EUI","devAddr":"00000000","fNwkSIntKey":"00000000000000000000000000000000","sNwkSIntKey":"00000000000000000000000000000000","nwkSEncKey":"00000000000000000000000000000000","appSKey":"00000000000000000000000000000000","fCntUp":0,"nFCntDown":0,"aFCntDown":0}}}'
```

#### Sample Response

```http
HTTP/1.1 201 Created
```

### Update End-Device Activation

Updates the activation for the given end-device.

#### Request Definition

PATCH /api/v4/end-devices/{dev_eui}/activation

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device

#### Request Headers

Header|Required|Description
---|---|---
`Content-Type: application/json`|true|
`Accept: application/json`|true|

#### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [Activation](#activation) object.

#### Response

Status|Body|Description
---|---|---
`201 OK`|A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing an [End-Device Activation](#end-device-activation)|Successful response
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request PATCH \
  --url 'https://application.lorawan.netemera.com/api/v4/end-devices/DEV_EUI/activation' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{"data":{"type":"end-device-activation","id":"DEV_EUI","attributes":{"devEui":"DEV_EUI","devAddr":"00000000","fNwkSIntKey":"00000000000000000000000000000000","sNwkSIntKey":"00000000000000000000000000000000","nwkSEncKey":"00000000000000000000000000000000","appSKey":"00000000000000000000000000000000","fCntUp":0,"nFCntDown":0,"aFCntDown":0}}}'
```

#### Sample Response

```http
HTTP/1.1 200 OK
```

### Delete End-Device Activation

Deletes the activation for the given EUI.

#### Request Definition

DELETE /api/v4/end-devices/{dev_eui}/activation

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Response

Status|Body|Description
---|---|---
`204 No Content`||The activation has been deleted
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`404 Not Found`||The application does not exist
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request DELETE \
  --url 'https://application.lorawan.netemera.com/api/v4/end-devices/DEV_EUI/activation' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample Response

```http
HTTP/1.1 204 No Content
```

## Uplink Packets

Uplink packets contain payload sent by commissioned end-devices along with radio and network metadata.

### Uplink Packet

An uplink packet has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`devEui`|string|false|The EUI-64 of the end-device
`recvTime`|string|false|The UTC timestamp of packet reception from the end-device in ISO 8601 format
`fPort`|integer|true|The port in the range from 1 to 223
`fCntUp`|integer|false|The uplink frame counter
`ack`|boolean|false|The acknowledgment of the last downlink packet (if any)
`adr`|boolean|false|The ADR flag. Positive value indicates that the end device is respecting ADR MAC commands sent by Network Server
`dataRate`|integer|false|The data rate in the range from 0 to 7
`ulFreq`|number|false|The radio frequency in Mhz, e.g., 868.10
`gwInfo`|[[Gateway Information](#gateway-information)]|true|An array of information objects about the gateways that received the packet. Sorted in descending order by RSSI. An uplink packet can be received by multiple gateways serving the area
`frmPayload`|string|true|An optional payload in hex

#### Gateway Information

Gateway information consists of radio metadata from a gateway that received the packet. The following attributes are available:

Attribute|Type|Optional|Description
---|---|---|---
`id`|string|false|The ID of the gateway
`rssi`|number|false|The RSSI
`snr`|number|false|The SNR

### Receive End-Device Uplink Packets

Opens a SSE stream of uplink packets received by the network from the given end device.

Supports [reconnections using the last received event ID](https://www.w3.org/TR/eventsource/#processing-model) to avoid packet loss by the client. Packet order is maintained.

The endpoint enables retrieving packets with timestamps in a given time range by specifying `filter[since]` and `filter[until]`.

#### Request Definition

GET /api/v4/uplink-packets/end-devices/{dev_eui}?filter[since]={since}&filter[until]={until}&filter[tail]={tail}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device
`filter[since]`|string|false|The UTC timestamp of the beginning of the time range in ISO 8601 format (inclusive)
`filter[until]`|string|false|The UTC timestamp of the end of the time range in ISO 8601 format (exclusive)
`filter[tail]`|number|false|The number of last rows to retrieve

#### Request Headers

Header|Required|Description
---|---|---
`Accept: text/event-stream`|true|Accept an SSE stream
`Last-Event-ID: {id}`|false|Receive packets from the point in the stream given by the `id` parameter
`Cache-Control: no-cache`|true|Disable caching

#### Response

Status|Body|Description
---|---|---
`200 OK`|An SSE stream of [Uplink Packet objects](#uplink-packet-object) with event IDs. Empty keep-alive messages are sent in case of no traffic|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample request

```shell
curl \
  --request GET \
  --no-buffer \
  --url 'https://application.lorawan.netemera.com/api/v4/uplink-packets/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: text/event-stream' \
  --header 'Last-Event-ID: LAST_EVENT_ID' \
  --header 'Cache-Control: no-cache'
```

#### Sample response

```http
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/event-stream

data:{
  "devEui": "DEV_EUI",
  "recvTime": "2016-10-31T21:41:51.765598114Z",
  "fPort": 1,
  "fCntUp": 1,
  "ack": false,
  "adr": false,
  "dataRate": 1,
  "ulFreq": 868.3,
  "gwInfo": [{
      "id": "7276ff002e0400e8",
      "rssi": -116.0,
      "snr": -15.2
  }],
  "frmPayload":"05F0"
}
id:AAABWByxABU=

data:

data:

data:{
  "devEui": "DEV_EUI",
  "recvTime": "2016-10-31T21:41:53.765598114Z",
  "fPort": 11,
  "fCntUp": 123,
  "ack": false,
  "adr": false,
  "dataRate": 0,
  "ulFreq": 868.2,
  "gwInfo": [{
      "id": "7276ff002e0400e8",
      "rssi": -80.0,
      "snr": -11.5
  }],
  "frmPayload":"AAF0"
}
id:AAABWByxB+U=
```

### Receive Application Uplink Packets

Opens a SSE stream of uplink packets received by the network from all end devices belonging to the given application.

Supports [reconnections using the last received event ID](https://www.w3.org/TR/eventsource/#processing-model) to avoid packet losses by the client. Packet order is maintained for each device (no total order within an application).

The endpoint enables retrieving packets with timestamps in a given time range by specifying `filter[since]` and `filter[until]`.

#### Request Definition

GET /api/v4/uplink-packets/applications/{application_id}?filter[since]={since}&filter[until]={until}&filter[tail]={tail}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`application_id`|string|true|The identifier of an application
`filter[since]`|string|false|The UTC timestamp of the beginning of the time range in ISO 8601 format (inclusive)
`filter[until]`|string|false|The UTC timestamp of the end of the time range in ISO 8601 format (exclusive)
`filter[tail]`|number|false|The number of last rows to retrieve

#### Request Headers

Header|Required|Description
---|---|---
`Accept: text/event-stream`|true|Accept an SSE stream
`Last-Event-ID: {id}`|false|Receive packets from the point in the stream given by the `id` parameter
`Cache-Control: no-cache`|true|Disable caching

#### Response

Status|Body|Description
---|---|---
`200 OK`|An SSE stream of [Uplink Packet objects](#uplink-packet-object) with event IDs. Empty keep-alive messages are sent in case of no traffic|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample request

```shell
curl \
  --request GET \
  --no-buffer \
  --url 'https://application.lorawan.netemera.com/api/v4/uplink-packets/applications/APPLICATION_ID' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: text/event-stream' \
  --header 'Last-Event-ID: LAST_EVENT_ID' \
  --header 'Cache-Control: no-cache'
```

#### Sample response

```http
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/event-stream

data:{
  "devEui": "70b3d5ca40000031",
  "recvTime": "2016-10-31T21:41:51.765598114Z",
  "fPort": 1,
  "fCntUp": 1,
  "ack": false,
  "adr": false,
  "dataRate": 0,
  "ulFreq": 868.3,
  "gwInfo": [{
      "id": "7276ff002e0400e8",
      "rssi": -116.0,
      "snr": -15.2
  }],
  "frmPayload":"200A"
}
id:AAABWByxABU=

data:

data:

data:{
  "devEui": "70b3d5ca40000032",
  "recvTime": "2016-10-31T21:41:53.765598114Z",
  "fPort": 11,
  "fCntUp": 123,
  "ack": false,
  "adr": false,
  "dataRate": 1,
  "ulFreq": 868.2,
  "gwInfo": [{
      "id": "7276ff002e0400e8",
      "rssi": -80.0,
      "snr": -11.5
  }],
  "frmPayload":"001F"
}
id:AAABWByxB+U=
```

## Downlink Packets

Downlink packets are sent by a client to commissioned end-devices.

### Downlink Packet

A Downlink Packet has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`devEui`|string|false|The EUI-64 of the end device
`recvTime`|string|true|The UTC timestamp of packet reception from the client in ISO 8601 format
`fPort`|integer|true|The port in the range from 1 to 223
`confirmed`|boolean|false|Require packet reception confirmation from the end device
`frmPayload`|string|true|The payload in hex

### Send End-Device Downlink Packet

Sends a downlink packet to the given end device. The packet will be queued by the network and sent to the end-device in the first available time window.

#### Request Definition

POST /api/v4/downlink-packets/end-devices/{dev_eui}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device

#### Request Headers

Header|Required|Description
---|---|---
`Accept: application/json`|true|

#### Request Body

A [JSON:API-style document](https://jsonapi.org/format/#document-structure) containing [Downlink Packet](#downlink-packet) attributes.

#### Response

Status|Body|Description
---|---|---
`202 Accepted`||The packet has been submitted
`401 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample Request

```shell
curl \
  --request POST \
  --url 'https://application.lorawan.netemera.com/api/v4/downlink-packets/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{"data":{"type":"downlink-packet":,"attributes":{"devEui":"DEV_EUI","fPort":2,"confirmed":false,"frmPayload":"0000"}}}'
```

#### Sample Response

HTTP:

```http
HTTP/1.1 201 Created
```

### Receive End-Device Downlink Packets

Opens a SSE stream of downlink packets received by the network from the client.

Supports [reconnections using the last received event ID](https://www.w3.org/TR/eventsource/#processing-model) to avoid packet loss by the client. Packet order is maintained.

The endpoint enables retrieving packets with timestamps in a given time range by specifying `filter[since]` and `filter[until]`.

#### Request Definition

GET /api/v4/downlink-packets/end-devices/{dev_eui}?filter[since]={since}&filter[until]={until}&filter[tail]={tail}

#### Request Parameters

Parameter|Type|Required|Description
---|---|---|---
`dev_eui`|string|true|The EUI-64 of the end-device
`filter[since]`|string|false|The UTC timestamp of the beginning of the time range in ISO 8601 format (inclusive)
`filter[until]`|string|false|The UTC timestamp of the end of the time range in ISO 8601 format (exclusive)
`filter[tail]`|number|false|The number of last rows to retrieve

#### Request Headers

Header|Required|Description
---|---|---
`Accept: text/event-stream`|true|Accept an SSE stream
`Last-Event-ID: {id}`|false|Receive packets from the point in the stream given by the `id` parameter
`Cache-Control: no-cache`|true|Disable caching

#### Response

Status|Body|Description
---|---|---
`200 OK`|An SSE stream of [Downlink Packet objects](#downlink-packet-object) with event IDs. Empty keep-alive messages are sent in case of no traffic|Successful response
`400 Bad Request`||Request validation failed
`401 Unauthorized`||Missing or invalid access token
`405 Forbidden`||Missing permissions

#### Sample request

```shell
curl \
  --request GET \
  --no-buffer \
  --url 'https://application.lorawan.netemera.com/api/v4/downlink-packets/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: text/event-stream' \
  --header 'Last-Event-ID: LAST_EVENT_ID' \
  --header 'Cache-Control: no-cache'
```

#### Sample response

```http
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/event-stream

data:{
  "devEui": "DEV_EUI",
  "recvTime": "2016-10-31T21:41:51.765598114Z",
  "fPort": 2,
  "confirmed":false,
  "frmPayload":"0000"
}
id:AAABWByxABU=

data:

data:

data:{
  "devEui": "DEV_EUI",
  "recvTime": "2016-10-31T22:34:01.000Z",
  "fPort": 2,
  "confirmed":false,
  "frmPayload":"0001"
}
id:AAABWByxABU=
```