> PROPOSAL ONLY

# Introduction

DCS has been designed to integrate seamlessly with other systems.
In order to provide this integration a JSON based Web API is
provided that allows programmatic access to metered data that has been collected by DCS.

# Authentication

## Modes

The API can be configured via DCSWebApp application settings in 2 modes:

- Authenticated

  All calls to the API must contain a security token which will be obtained by the sign-in method (see xxx). The credentials used to sign-in in will correspond to normal DCS user account.

  When called in Authenticated mode restrictions applied with respect to call rate and the quantity of data requested.

- Unauthenticated

  Calls to the API do not need to contain a security token so no sign-in is required. All calls to the API will use a default DCS user account that can be configured via DCSWebApp application settings.

  When called in Unauthenticated mode more severe restrictions are applied with respect to call rate and the quantity of data requested.

> It is possible to configure the API with both modes simultaneously. The URL used for each mode will be different.

> By default the public API will be disabled.

## Authenticated mode

In this mode all calls to the Web API require an authentication token to be
sent in a cookie as part of the HTTP request.

In order to obtain an authentication token the client must send a
POST message providing valid DCS credentials to the server as a JSON object
in the request body.

### Request

```
POST /api/Account/login
```

### JSON Parameters

| Name     | Value        | Note |
| -------- | ------------ | ---- |
| username | DCS username |
| password | DCS password |

### Response

If the credentials are correct the server will respond with
response code 200 and a cookie containing an authentication
token. The cookie name will start with COHERENT-DCSV3.

### Sample

```
POST https://www.coherent-research.co.uk/DCS/api/account/login
content-type: application/json

{"username": "user", "password": "mypassword" }

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Set-Cookie: COHERENT-DCSV3 ...

{
  "username": "user",
  "role": "Operator",
  "appVersion": "3.0.0"
}
```

The cookie should be sent by the client to the
server in all subsequent HTTP messages. The information returned in
the response body is for information only.

> The authentication cookie will have an expiration time after which
> it will no longer be accepted by the server and the server will respond
> to all requests with response code 401. In this case the login
> message must be repeated to obtain a new authentication token.

Once authenticated the caller can access the API via the URL:

```
https://HOSTNAME/DCS/api/METHOD
```

## Unauthenticated mode

In this mode no sign-in is required and the caller can access the API via the URL:

```
https://HOSTNAME/DCS/api/public/METHOD
```

# Queries

## Requests

All queries to DCS will use a HTTP GET request message and the parameters will be sent as part of the URL
as a query string in the form:

```
GET URL?param1=value1&param2=value2
```

## Response codes

Standard HTTP response codes are used in the response to all queries.
The following codes are used

| Response code             | Meaning                                                                                                                                                                                                                |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200 OK                    | Indicates that the query was successful and the response body will contain a JSON object (or array of objects) with the requested data.                                                                                |
| 400 Bad Request           | Indicates the request is not valid or understood by DCS. The body of the response will provide more details of why the request is considered bad.                                                                      |
| 401 Unauthorized          | Indicates that the caller has not provided a valid authentication token. See above                                                                                                                                     |
| 403 Forbidden             | Indicates that the caller does not have the appropriate authority to perform the request (even though the authentication token is valid). This implies that the DCS account being used does not have the correct role. |
| 429 Too Many Requests     | Indicates that the caller has sent too many requests in a given amount of time.                                                                                                                                        |
| 500 Internal Server Error | Indicates a fault on the server. Contact the DCS administrator.                                                                                                                                                        |

# Meters and Virtual Meters

## Get meters

Returns a list of all meters defined in DCS including the registers for each meter.

### Request

```
GET /api/meters
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
| remoteAddress    | Address used to communicate with meter, e.g. IP address, modem number | Form depends on connectionMethod                                                                                                                                                            |
| serialNumber     | Manufacturer's meter serial number                                    | Form depends on type of meter                                                                                                                                                               |
| status           | Online, offline, disabled, unknown                                    | Online/offline status is determine by the age of the latest reading compared to what would be expected for the connectionMethod. Disabled indicates that data collection has been disabled. |
| registers        | Array of register objects                                             | See below                                                                                                                                                                                   |

Register objects

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
GET https://www.coherent-research.co.uk/DCS/api/meters
Cookie: COHERENT-DCSV3...

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
GET /api/virtualMeters
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

Register alias objects

| Property   | Value                                          | Note                                |
| ---------- | ---------------------------------------------- | ----------------------------------- |
| alias      | String used to refer to register in expression |
| registerId | Register id                                    | Id for register the alias refers to |

### Sample

Note that some properties have been removed for simplicity.

```
GET https://www.coherent-research.co.uk/DCS/api/virtualMeters
Cookie: COHERENT-DCSV3...

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

# Get Metered Data

Returns an array of readings for the specified register or virtual meter and timespan.

## Request

```
GET api/getmetereddata?QUERYSTRING
```

## Query String Parameters

| Name         | Value                                                                                                                      | Note                                                                                                                                                                                                                |
| ------------ | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id           | Id of the register or virtual meter in the form Rx or VMx where x corresponds to the register's or virtual meter's DCS ID. | Required. If R and VM are omitted the id is assumed to refer to a DCS register ID.                                                                                                                                  |
| format       | Specifies the format of the data that will be returned. The options are: standard, expanded                                | Optional, default is _standard_. See below for more details.                                                                                                                                                        |
| dateTime     | Start date/time in ISO 8601 format.                                                                                        | Required. Note that the start date/time must be consistent with the IntegrationPeriod, e.g. if the integration period is set specified as 'week' **dateTime** must correspond to the start of a calendar week.      |
| periodCount  | The number of periods.                                                                                                     | Optional. Alternatively the **end** parameter can be specified. Either **periodCount** or **end** must be specified.                                                                                                |
| end          | End date/time (exclusive) in ISO 8601 format.                                                                              | Required. Note that the start date/time must be consistent with the IntegrationPeriod.                                                                                                                              |
| periodType   | halfHour, hour, day, week, month                                                                                           | Optional, default is halfHour.                                                                                                                                                                                      |
| calibrated   | false or true                                                                                                              | If set any Calibration Readings associated with the register will be used to adjust the TotalValues. Optional, default is true                                                                                      |
| interpolated | false or true                                                                                                              | If set DCS will attempt to estimate values for any missing data. Optional, default is true                                                                                                                          |
| source       | automatic, manual, merged                                                                                                  | In DCS automatically collected readings can be supplemented with manual readings and this property can indicate whether to return use manual readings. Default value is Automatic and this should normally be used. |

## Response

The format of the data returned in the response message depends ion the _format_ parameter.

### format = standard

A single object the contains the following properties:

| Property            | Value                                                 | Note                                                                                                      |
| ------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| startDateTime       | Start of range                                        | Corresponds to the **dateTime** parameter in the request                                                  |
| endDateTime         | End of range                                          | Corresponds to the **end** parameter in the request (or calculated from **dateTime** and **periodCount**) |
| meter               | The name of the name of the register or virtual meter | The register name will be the full register name, i.e. METER NAME: REGISTER NAME                          |
| periodType          | halfHour, hour, day, week, month                      | Corresponds to the **periodType** parameter in the request                                                |
| startMeterReading   | Total reading value at startDateTime                  |
| endMeterReading     | Total reading value at endDateTime                    |
| totalConsumption    | Total consumption over the requested period           |
| consumptionByPeriod | See table below                                       |

**consumptionByPeriod** is an array of objects each of which corresponds to a single reading. Each object has the following properties:
| Property | Value | Note |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| startDateTime | Start of period timestamp | ISO 8601 format
| value | Consumption for the period | If the register is instantaneous this will always be 0 |
| isPadded | Specifies if the value was generated to pad out missing data |

### format = expanded

An array of objects each of which corresponds to a single reading. Each object has the following properties:

| Property       | Value                                                        | Note                                                                           |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| startTime      | Start of period timestamp                                    |
| duration       | Period length in minutes                                     | Corresponds to integrationPeriod in query                                      |
| totalValue     | Register total at startTime                                  |
| periodValue    | Consumption for the period                                   | If the register is instantaneous this will always be 0                         |
| isGenerated    | Specifies if the value was generated to pad out missing data |
| isInterpolated | Specifies if the value was estimated by DCS                  | Corresponds to interpolated in query (IsGenerated is undefined if this is set) |

### Samples

Authenticated mode, format = expanded

```
GET https://www.coherent-research.co.uk/DCS/api/getmetereddata?id=r100 ...
       &dateTime=2019-02-01&end=2019-02-01&periodType=hour
Cookie: COHERENT-DCSV3...

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

[
  {
    "id": 26167735,
    "startTime": "2019-02-01T00:00:00Z",
    "duration": 30,
    "totalValue": 10161.0,
    "periodValue": 10.0,
    "isGenerated": false,
    "isInterpolated": false
  },
  {
    "id": 26167736,
    "startTime": "2019-02-01T00:30:00Z",
    "duration": 30,
    "totalValue": 10172.0,
    "periodValue": 5.0,
    "isGenerated": false,
    "isInterpolated": false
  },
  ...
]
```

Unauthenticated mode, format = standard

```
GET https://www.coherent-research.co.uk/DCS/api/getmetereddata?id=r100 ...
       &dateTime=2021-09-20T18:00:00Z&periodCount=4&periodType=halfHour

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "consumptionByPeriod":
    [
      {
        "isPadding":false,
        "startDateTime":"2021-09-20T18:00:00Z",
        "value":9.0400000000008731
      },
      {
        "isPadding":false,
        "startDateTime":"2021-09-20T18:30:00Z",
        "value":8.39999999999418
      },
      {
        "isPadding":false,
        "startDateTime":"2021-09-20T19:00:00Z",
        "value":8.0800000000017462
      },
      {
        "isPadding":false,
        "startDateTime":"2021-09-20T19:30:00Z",
        "Value":0.0
      }
    ],
  "endDateTime":"2021-09-20T20:00:00Z",
  "endMeterReading":3686343.96,
  "meter":"Elec",
  "periodType":"HalfHour",
  "startDataTime":"2021-09-20T18:00:00Z",
  "startMeterReading":3686318.44,
  "totalConsumption":25.5199999999968,

```

## Restrictions and rate limiting

TODO

## Configuration

The public API are configured by changing the settings in the DCSWebAppSettings file.
