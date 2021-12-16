> PROPOSAL ONLY

# Introduction

DCS has been designed to integrate seamlessly with other systems.
In order to enable this integration a JSON based Web API is
provided that allows programmatic access to metered data that has been collected by DCS.

The API is deployed as a separate application (DcsWebApi) in IIS. It is envisaged that this will be deployed in the same instance of IIS that hosts DcsWebApp but that is not a requirement. DcsWebApi must have access to the DCS Database but otherwise it does not interact with other DCS components.

# Authentication

## Modes

The API has two modes:

- Authenticated

  All calls to the API must contain a security token which will be
  obtained by the sign-in method (see below).
  The credentials used to sign-in in will correspond to a normal DCS user account (managed by DcsWebApp).

- Unauthenticated

  Calls to the API do not need to contain a security token so no
  sign-in is required. All calls to the API will use a default DCS user account.
  The account corresponds to a normal DCS user account (managed by DcsWebApp) and the actual account that is used is configurable in the DcsWebApi application.

> It is possible to configure the API with either mode or both modes simultaneously. The URL used for each mode will be different.

## Authenticated mode

In this mode all calls to the Web API require an authentication token to be
sent in a cookie in the HTTP request headers.

In order to obtain an authentication token the client must send a
POST message providing valid DCS credentials to the server as a JSON object
in the request body.

### Sign in

#### Request

```
POST /authentication/signin
```

#### JSON Parameters

| Name     | Value        | Note |
| -------- | ------------ | ---- |
| username | DCS username |
| password | DCS password |

#### Response

If the credentials are correct the server will respond with
response code 200 and a cookie containing an authentication
token. The cookie name will start with COHERENT-DCS-API.

#### Sample

```
POST https://www.coherent-research.co.uk/DCSWebApi/authentication/signin
content-type: application/json

{"username": "user", "password": "mypassword" }

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Set-Cookie: COHERENT-DCS-API ...

{
  "username": "user",
  "role": "Operator",
  "appVersion": "3.0.0"
}
```

The cookie should be sent by the client to the
server in all subsequent HTTP messages. The information returned in
the response body is for information only.

> The authentication cookie will have a configurable expiration time after which
> it will no longer be accepted by the server and the server will respond
> to all requests with response code 401. In this case the login
> message must be repeated to obtain a new authentication token.

Once authenticated the caller can access the API via the URL:

```
https://HOSTNAME/DCSWebApi/METHOD
Accept: application/json
Cookie: COHERENT-DCS-API...
```

### Sign out

A user may optionally call the signout method at the end of a session. This will have the effect
of invalidating cancelling the token so it can not be used again. It will also return an empty cookie.

#### Request

```
POST /authentication/signout

HTTP/1.1 204
Cookie: COHERENT-DCS-API... = ""
```

#### Sample

```
POST https://www.coherent-research.co.uk/DCSWebApi/authentication/signout
content-type: application/json

HTTP/1.1 204 No Content
```

#### Response

The server will respond with
response code 204 in all cases. If the request contains a valid authentication
token it will be cancelled making it no longer valid.

## Unauthenticated mode

In this mode no sign-in is required and the caller can access the API via the URL:

```
https://HOSTNAME/DCSWebApi/public/METHOD
Accept: application/json
```

# Queries

## Requests

All queries will use a HTTP GET request message and the parameters will be sent as part of the URL as a query string in the form:

```
GET URL?param1=value1&param2=value2
Accept: application/json
```

## Response codes

Standard HTTP response codes are used in the response to all queries.
The following codes are used

| Response | Meaning               | Use                                                                                                                                                                                           |
| -------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200      | OK                    | Indicates that the query was successful and the response body will contain a JSON object (or array of objects) with the requested data.                                                       |
| 400      | Bad Request           | Indicates the request is not valid or understood by DCS. The body of the response will provide more details as to why the request is considered bad.                                          |
| 401      | Unauthorized          | Indicates that the caller has not provided a valid authentication token (if required). See above                                                                                              |
| 403      | Forbidden             | Indicates that the caller does not have the appropriate authority to perform the request (even though the authentication token is valid). The body of the response will provide more details. |
| 429      | Too Many Requests     | Indicates that the caller has sent too many requests in a given amount of time.                                                                                                               |
| 500      | Internal Server Error | Indicates a fault on the server. This should be considered a DCS error and raise the issue with the server administrator                                                                      |

For each error response (i.e. where the response code does not equal 200) the payload will be a JSON object with the following properties:

| Property | Value                     | Note                                 |
| -------- | ------------------------- | ------------------------------------ |
| details  | Reason the request failed | This will be a human readable string |

In certain cases extra fields may be added to the response object (e.g. see the chapter on Rate limiting below).

## Future compatibility

It is envisaged that the response payloads may gain extra properties over time. Therefore to
assist in maintaining future compatibility the consumers of the service should silently ignore any properties in the response payloads that are recognised.

# Meters and Virtual Meters

## Get meters

Returns a list of all meters defined in DCS including the registers for each meter.

### Request

```
GET /meters
```

Returns an array of meter objects. Each meter object will contain 1 or more
register objects.

### Response

An array of meter objects with the following properties.
Note that only a subset of the properties are documented here.

| Property         | Value                                                                 | Note                                                                                                                                                                                        |
| ---------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id               | Unique meter id                                                       | Unsigned 32 integer                                                                                                                                                                         |
| connectionMethod | tcp, modem, idc                                                       | How DCS connects to collect data                                                                                                                                                            |
| deviceId         | Outstation address                                                    | Form depends on type of meter                                                                                                                                                               |
| name             | Unique meter name                                                     |
| meterType        | Meter type name                                                       |
| remoteAddress    | Address used to communicate with meter, e.g. IP address, modem number | Form depends on connectionMethod                                                                                                                                                            |
| serialNumber     | Manufacturer's meter serial number                                    | Form depends on type of meter                                                                                                                                                               |
| mpn              | Metering point number                                                 | Will be included if set in DCS                                                                                                                                                              |
| service          | Meter service                                                         | Will be included if set in DCS                                                                                                                                                              |
| status           | Online, offline, disabled, unknown                                    | Online/offline status is determine by the age of the latest reading compared to what would be expected for the connectionMethod. Disabled indicates that data collection has been disabled. |
| registers        | Array of register objects                                             | See below                                                                                                                                                                                   |

**Register objects**

| Property           | Value                                                            | Note                             |
| ------------------ | ---------------------------------------------------------------- | -------------------------------- |
| id                 | Unique register id                                               | Unsigned 32 integer              |
| address            | Register address                                                 | Form depends on type of meter    |
| isInstantaneous    | Quantity is instantaneous (e.g. W) or cumulative (e.g. Wh)       | Boolean                          |
| name               | Register name                                                    | Unique for a given meter         |
| scaleFactor        | Scale factor applied to readings when read                       | May be empty                     |
| defaultScaleFactor | Default scale factor applied to readings if scaleFactor is empty | May be empty implying no scaling |
| unit               | Unit for readings                                                |

### Sample

Note that some properties have been removed for simplicity.

```
GET https://www.coherent-research.co.uk/DCSWebApi/meters
Accept: application/json
Cookie: COHERENT-DCS-API...

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

[
  {
    "connectionMethod": "tcp",
    "deviceId": "",
    "id": 405,
    "name": "Sample meter",
    "remoteAddress": "10.8.222.99:5000",
    "serialNumber": "R1100729",
    "status": "online",
    "registers": [
      {
        "address": "130",
        "id": 733,
        "isInstantaneous": false,
        "name": "Active Energy (import)",
        "scaleFactor": "",
        "defaultScaleFactor": "1",
        "unit": "kWh"
      }
    ],
  },
  ...
}]
```

## Get virtual meters

Returns a list of all virtual meters defined in DCS. Each virtual meter object will contain 1 or more
register alias objects.

### Request

```
GET /virtualMeters
```

Returns an array of virtual meter objects.

### Response

An array of virtual meter objects with the following properties.
Note that only a subset of the properties are documented here.

| Property        | Value                                                                                   | Note                                                                                      |
| --------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| id              | Unique meter id                                                                         | Unsigned 32 integer                                                                       |
| name            | Unique meter name                                                                       |
| expression      | Mathematical expression used to calculate values as a function of one or more registers |
| isInstantaneous | Quantity is instantaneous (e.g. W) or cumulative (e.g. Wh)                              | Boolean                                                                                   |
| unit            | Unit for readings                                                                       |
| registerAliases | Array of register alias objects                                                         | A register alias refers to a register in a meter by a name used in the express. See below |

**Register alias objects**

| Property   | Value                                          | Note                                |
| ---------- | ---------------------------------------------- | ----------------------------------- |
| alias      | String used to refer to register in expression |
| registerId | Register id                                    | Id for register the alias refers to |

### Sample

Note that some properties have been removed for simplicity.

```
GET https://www.coherent-research.co.uk/DCSWebApi/virtualMeters
Accept: application/json
Cookie: COHERENT-DCS-API...

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

[
   {
    "id": 9,
    "name": "Sample Virtual Meter",
    "expression": "R1+R2+R3",
    "decimalPlaces": 2,
    "isInstantaneous": false,
    "unit": "kWh",
    "registerAliases": [
      {
        "alias": "R1",
        "registerId": 1156,
        "registerName": ""
      },
      {
        "alias": "R2",
        "registerId": 1164,
        "registerName": ""
      },
      {
        "alias": "R3",
        "registerId": 1168,
        "registerName": ""
      }
    ]
  },
  ...
}]
```

# Get Readings

Returns an array of readings for the specified register or virtual meter and timespan.

## Request

```
GET readings?QUERYSTRING
```

## Query String Parameters

| Name         | Value                                                                                                                      | Note                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------ | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| id           | Id of the register or virtual meter in the form Rx or VMx where x corresponds to the register's or virtual meter's DCS ID. | Required.                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| format       | Specifies the format of the data that will be returned. The options are: standard                                          | Optional, default is _standard_. More options may be added in the future.                                                                                                                                                                                                                                                                                                                                                                                          |
| startTime    | Start date/time as UTC in the format yyyy-MM-ddTHH:mm:ssZ, e.g. 2021-06-231T22:30:00Z                                      | Optional. See not below. If included **startTime** must be consistent with **periodType**, e.g. if the period type is set specified as _week_ **startTime** must correspond to the start of a calendar week.                                                                                                                                                                                                                                                       |
| periodCount  | The number of periods.                                                                                                     | Optional. See note below.                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| endTime      | End date/time as UTC in the format yyyy-MM-ddTHH:mm:ssZ, e.g. 2021-06-231T23:30:00Z                                        | Optional. See not below. . If included **endTime** must be consistent with **periodType**, e.g. if the period type is set specified as _week_ **endTime** must correspond to the end of a calendar week.                                                                                                                                                                                                                                                           |
| periodType   | halfHour, hour, day, week, month                                                                                           | Optional, default is halfHour. If the register or virtual meter is instantaneous the periodType can not be set to a value other than halfHour.                                                                                                                                                                                                                                                                                                                     |
| calibrated   | true or false                                                                                                              | If set to true any Calibration Readings associated with the register will be used to adjust the TotalValues. Optional, default is false. Since calibration is not supported for instantaneous registers or virtual meters this parameter will be ignored in those cases.                                                                                                                                                                                           |
| interpolated | true or false                                                                                                              | If set to false the returned data will only contain the readings that exist in DCS. If set to true DCS will attempt to estimate values for any missing data. If for any reason DCS is unable to interpolate the missing readings the returned data will only contain the readings that exist in DCS. Optional, default is false. Since interpolation is not supported for instantaneous registers or virtual meters this parameter will be ignored in those cases. |

> The timespan covered by the request can be specified by including any 2 of **startTime**, **endTime** or **periodCount**.

E.g. the, assuming the periodType='day' the
timespan from midnight on 1st of January 2000 to midnight on the 1st of February 2000 can be specified in either of the following 3 ways::

```
startTime: 2000-01-01T00:00:00Z
endTime: 2000-02-01T00:00:00Z
```

```
startTime: 2000-01-01T00:00:00Z
periodCount: 31
```

```
endTime: 2000-02-01T00:00:00Z
periodCount: 31
```

## Response

The format of the data returned in the response message depends ion the _format_ parameter.

### format = standard

A single object that contains the following properties:

| Property        | Value                                                 | Note                                                                                                                                                                                                                                        |
| --------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| startTime       | Start of range                                        | Corresponds to the start of the timespan as specified by the combination of **startTime**, **endTime**, **periodType** and **periodCount** parameters in the request.                                                                       |
| endTime         | End of range                                          | Corresponds to the end of the timespan as specified by the combination of **startTime**, **endTime**, **periodType** and **periodCount** parameters in the request.                                                                         |
| name            | The name of the name of the register or virtual meter | The register name will be the full register name, i.e. METER NAME: REGISTER NAME                                                                                                                                                            |
| periodType      | halfHour, hour, day, week, month                      | Corresponds to the **periodType** parameter in the request                                                                                                                                                                                  |
| unit            | The unit for each value, e.g. kWh                     |                                                                                                                                                                                                                                             |
| readingDuration | The duration for each reading.                        | If the reading represents a meter total, or an instantaneous value this will be 0. If the reading represents consumption over time the duration will correspond to the periodType (this will normally only be the case for virtual meters). |
| readings        | An array of reading objects. See table below          |

A **reading** object corresponds to a single reading and has the following properties:

| Property  | Value                                               | Note      |
| --------- | --------------------------------------------------- | --------- |
| timestamp | The UTC time corresponding to the reading.          |           |
| value     | The value of the reading                            |           |
| status    | A set of flags indicating the status of the reading | See below |

The **status** is an integer that in made up of one or more of the following flags logically ORed together. If no flags are present the status parameter will be 0.

| Flag         | Value | Meaning                                                                                                              |
| ------------ | ----- | -------------------------------------------------------------------------------------------------------------------- |
| interpolated | 1     | No corresponding reading exists in DCS and this value has been interpolated by DCS                                   |
| reset        | 2     | The value corresponds to a discontinuity in the metered data as indicated by the presence of a Register Reset in DCS |

### Samples

Authenticated mode, format = standard

```
GET https://www.coherent-research.co.uk/DCSWebApi/metereddata?id=r100 ...
       &startTime=2021-09-20T18:00:00Z&periodCount=4&periodType=halfHour
Accept: application/json
Cookie: COHERENT-DCS-API...

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{

 "endTime": "2021-09-20T20:00:00Z",
 "name": "Electricity: Active Energy Import",
 "periodType": "halfHour",
 "startTime": "2021-09-20T18:00:00Z",
 "duration": 0,
 "unit": "kWh",
 "readings":
    [
      {
        "status":0,
        "timestamp":"2021-09-20T18:00:00Z",
        "value":9.04
      },
      {
        "status":1,
        "timestamp":"2021-09-20T18:30:00Z",
        "value":18.04
      },
      {
        "status":1,
        "timestamp":"2021-09-20T19:00:00Z",
        "value":43.08
      },
      {
        "status":0,
        "timestamp":"2021-09-20T19:30:00Z",
        "Value":59.0
      }
    ],

```

## Rate limiting

Rate limiting can be applied to the Authenticated and the Unauthenticated endpoints independently.

For each endpoint it is possible to specify the maximum number of requests per time period, e.g. 100 requests per minute, or 10 requests per second.

When the rate is exceeded the request will be rejected with the HTTP status code 429. The following property contents of the response will contain a JSON object with the following properties

| Property          | Value                                                          | Note       |
| ----------------- | -------------------------------------------------------------- | ---------- |
| backOffSuggestion | A suggested period that the caller should wait before retrying | In seconds |

> Rate limiting applies to an endpoint and not a client, in other words the rate limit applies to requests to an endpoint from **all** users.

## Query restrictions

Irrespective of whether the Authenticated or Unauthenticated mode the caller will be limited to a maximum date range when calling the GetMeteredData method. The size of the date range will be configurable for each mode separately.

If the requested data range exceeds the allowed range the request will be rejected with the HTTP status code 400 along with an appropriate error message.

## User restrictions

Irrespective of whether the Authenticated or Unauthenticated mode is used each call will be associated with a normal DCS user account. For the Authenticated mode the user account will correspond to the user name specified during the login, while for the Unauthenticated mode the user account will be the default account configured for this purpose.

In either case DCS will apply the normal users restrictions to the API call, i.e. the Meter Restriction profile will apply as normal and user restrictions such as expiration date, access start and end dates etc will apply as normal.

If the caller is restricted from access the requested data the request will be rejected with the HTTP status code 403 along with an appropriate error message.

## Configuration

All configurable options for the API are configured by changing the settings in the DCSWebApi Settings file. This file is a JSON file called appsettings.json which can be found in the directory when DcsWebApi is stalled.

Details to come.

| Property                  | Value                                                                 | Note                                                                             |
| ------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| privateEndPointEnabled    | true or false                                                         | Specifies if the authenticated end point is enabled. Optional, default = false   |
| privateEndPointRateLimit  | Maximum number of requests per second                                 |
| privateEndPointRangeLimit | Maximum timespan a metered data request can cover as a number of days | If set to 0 or omitted no limit will be applied.                                 |
| publicEndPointEnabled     | true or false                                                         | Specifies if the unauthenticated end point is enabled. Optional, default = false |
| publicEndPointRateLimit   | Maximum number of requests per second                                 |
| publicEndPointRangeLimit  | Maximum timespan a metered data request can cover as a number of days | If set to 0 or omitted no limit will be applied.                                 |
| publicEndPointAccount     | User account that will be used for the unauthenticated end point      |                                                                                  |
