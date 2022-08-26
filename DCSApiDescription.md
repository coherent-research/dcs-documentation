> DEPRECATED. See [replacement](DcsPublicApiDescription.md)

# Introduction

DCS has been designed to integrate seamlessly with other systems.
In order to provide this integration a JSON based Web API is
provided that allows programmatic access to
metered data that has been collected by DCS.

# Basics

## Authentication

All calls to the Web API require an authentication token to be
sent in a cookie as part of the HTTP request.

In order to obtain an authenication token the client must send a
POST message providing valid DCS credentials to the server as a JSON object
in the request body.

> **Note that all credentials and tokens are parsed between the server and client as JSON text strings, therefore the use of HTTPS rather than plain HTTP is strongly recommended for all transactions.**

### Request

```
POST /api/Account/login
```

### JSON Parameters

| Name     | Value        |
| -------- | ------------ |
| username | DCS username |
| password | DCS password |

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
  "role": "Operator"
}
```

The cookie should be sent by the client to the
server in all subsequent HTTP messages. The information returned in
the response body is for information only.

> The authentication cookie will have an expiration time after which
> it will no longer be accepted by the server and the server will respond
> to all requests with response code 401. In this case the login
> message must be repeated to obtain a new authentication token.

## Queries

### Requests

All queries to DCS will use a HTTP GET request message and the parameters will be sent as part of the URL
as a query string in the form:

```
GET URL?param1=value1&param2=value2
```

### Response codes

Standard HTTP response codes are used in the response to all queries.
The following codes are used

| Response code             | Meaning                                                                                                                                                                                                              |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200 OK                    | Indicates that the query was successful and the response body will contain a Json object (or array of objects) with the requested data.                                                                              |
| 400 Bad Request           | Indicates the request is not valid or understood by DCS. The body of the response will provide more details of why the request is considered bad.                                                                    |
| 401 Unauthorized          | Indicates that the user has not provided a valid authentication token. See above                                                                                                                                     |
| 403 Forbidden             | Indicates that the user does not have the appropriate authority to perform the request (even though the authentication token is valid). This implies that the DCS account being used does not have the correct role. |
| 500 Internal Server Error | Indicates a fault on the server. Contact the DCS administrator.                                                                                                                                                      |

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

# Metered Data

## Get register readings

Returns an array of readings for the specified register and timespan.

### Request

```
GET api/registerReadings/list?QUERYSTRING
```

### Query String Parameters

| Name              | Value                                        | Note                                                                                                                                                                                                                                                                                                          |
| ----------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| registerId        | Id of the register                           | Required                                                                                                                                                                                                                                                                                                      |
| start             | Start date in the form yyyy-MM-dd.           | Note that when specifying IntegrationPeriod of Week or Month the StartDate must correspond to the start of a calendar week or calendar month respectively.                                                                                                                                                    |
| end               | End date (exclusive) in the form yyyy-MM-dd. | The end date is optional. If not supplied it is assumed to have a value dependent on the start date and the IntegrationPeriod: HalfHour, Hour, Day: end date is set to return readings for 1 day. Week: end date is set to return readings for 1 week. Month: end date is set to return readings for 1 month. |
| integrationPeriod | halfHour, hour, day, week, month             | Optional, default is halfHour.                                                                                                                                                                                                                                                                                |
| useLocalTime      | false or true                                | If false all times will are in GMT, if true all time are local time (i.e. GMT/BST). Optional, default is false.                                                                                                                                                                                               |
| decimalPlaces     | 1-5                                          | Optional, default = 1                                                                                                                                                                                                                                                                                         |
| calibrated        | false or true                                | If set any Calibration Readings associated with the register will be used to adjust the TotalValues. Optional, default is false                                                                                                                                                                               |
| interpolated      | false or true                                | If set DCS will attempt to estimate values for any missing data.                                                                                                                                                                                                                                              |
| source            | automatic, manual, merged                    | In DCS automatically collected readings can be supplemented with manual readings and this property can indicate whether to return use manual readings. Default value is Automatic and this should normally be used.                                                                                           |

### Response

An array of objects with the following properties

| Property       | Value                                                        | Note                                                                           |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| startTime      | Start of period timestamp                                    |
| duration       | Period length in minutes                                     | Corresponds to intergrationPeriod in query                                     |
| totalValue     | Register total at startTime                                  |
| periodValue    | Consumption for the period                                   | If the register is instantaneous this will always be 0                         |
| isGenerated    | Specifies if the value was generated to pad out missing data |
| isInterpolated | Specifies if the value was estimated by DCS                  | Corresponds to interpolated in query (IsGenerated is undefined if this is set) |

### Sample

```
GET https://www.coherent-research.co.uk/DCS/api/registerReadings/list?registerId=100 ...
       &start=2019-02-01&integrationPeriod=hour
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

## Get virtual meter readings

Returns an array of readings for the specified virtual meter and timespan.

### Request

```
GET api/virtualMeterReadings/list?QUERYSTRING
```

### Query String Parameters

| Name              | Value                                        | Note                                                                                                                                                                                                                                                                                                          |
| ----------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| virtualMeterId    | Id of the virtualMeter                       | Required                                                                                                                                                                                                                                                                                                      |
| start             | Start date in the form yyyy-MM-dd.           | Note that when specifying IntegrationPeriod of Week or Month the StartDate must correspond to the start of a calendar week or calendar month respectively.                                                                                                                                                    |
| end               | End date (exclusive) in the form yyyy-MM-dd. | The end date is optional. If not supplied it is assumed to have a value dependent on the start date and the IntegrationPeriod: HalfHour, Hour, Day: end date is set to return readings for 1 day. Week: end date is set to return readings for 1 week. Month: end date is set to return readings for 1 month. |
| integrationPeriod | halfHour, hour, day, week, month             | Optional, default is halfHour.                                                                                                                                                                                                                                                                                |
| useLocalTime      | false or true                                | If false all times will are in GMT, if true all time are local time (i.e. GMT/BST). Optional, default is false.                                                                                                                                                                                               |
| decimalPlaces     | 1-5                                          | Optional, default = 1                                                                                                                                                                                                                                                                                         |
| calibrated        | false or true                                | If set any Calibration Readings associated with the register will be used to adjust the TotalValues. Optional, default is false                                                                                                                                                                               |
| interpolated      | false or true                                | If set DCS will attempt to estimate values for any missing data.                                                                                                                                                                                                                                              |
| source            | automatic, manual, merged                    | In DCS automatically collected readings can be supplemented with manual readings and this property can indicate whether to return use manual readings. Default value is Automatic and this should normally be used.                                                                                           |

### Response

An array of objects with the following properties

| Property       | Value                                                        | Note                                                                           |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| startTime      | Start of period timestamp                                    |
| duration       | Period length in minutes                                     | Corresponds to intergrationPeriod in query                                     |
| totalValue     | Total at startTime                                           | Note that this is always 0 for virtual meters                                  |
| periodValue    | Consumption for the period                                   | If the virtual meter is instantaneous this will always be 0                    |
| isGenerated    | Specifies if the value was generated to pad out missing data |
| isInterpolated | Specifies if the value was estimated by DCS                  | Corresponds to interpolated in query (IsGenerated is undefined if this is set) |

### Sample

```
GET https://www.coherent-research.co.uk/DCS/api/virtualMeterReadings/list?virtualMeterId=99 ...
       &start=2019-02-01&integrationPeriod=hour
Cookie: COHERENT-DCSV3...

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

[
  {
    "id": 26167735,
    "startTime": "2019-02-01T00:00:00Z",
    "duration": 30,
    "totalValue": 0,
    "periodValue": 10.0,
    "isGenerated": false,
    "isInterpolated": false
  },
  {
    "id": 26167736,
    "startTime": "2019-02-01T00:30:00Z",
    "duration": 30,
    "totalValue": 0,
    "periodValue": 5.0,
    "isGenerated": false,
    "isInterpolated": false
  },
  ...
]
```
