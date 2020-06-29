# S Rest API
A solution to consume and post REST API within Siemens S7 PLCs.

## Contents

1. [Overview](#Overview)
1. [Rest API Breakdown](#REST-API-Breakdown)
1. [JSON Handling](#JSON-Handling)
1. [REST Library](#REST-Library)
1. [Integration](#Integration)
1. [Extra Documentation](#Extra-Documentation)

## Overview

The [LHTTP](https://support.industry.siemens.com/cs/document/109763879/library-for-http-communication-(lhttp)?dti=0&lc=en-WW) library has been used as a starting point for making HTTP requests and receiving data. It's been extended to consume REST API. Two additional methods, PUT and DELETE, have been added and several features removed, to enable backwards-compatibility with S7-1200 PLCs. These were mainly the SSL security protections and the DNS domain resolution.

The code has been designed and tested on SIMATIC S7-1500 CPU 1516-3 PN/DP, Article 6ES7516-3AN01-0AB0. It uses dynamic array sizes which are only supported for firmware version **>= 2.0** on S7-1500 and firmware version **>= 4.2** on S7-1200. It has been compiled on SIMATIC S7-1200 1214FC DC/DC/DC, Article 6ES7 214-1AF40-0XB0 but it hasn't been tested due to unavailability of the device on PLCSIM Advanced.

The Library was designed with straightforward requests in mind. It **does not** support:
* Decoding or Encoding JSON content. The data is stored as a char array and no serialization is supported
* Authentication standards beyond basic. These have to be provided as a hashed username and password

## REST API Breakdown

From a functional perspective there's no difference between an HTTP and a REST request beyond standard conventions. In fact, REST communicates via the http protocol so in a sense it's a simple extension of what's already implemented.

For a detailed breakdown of the HTTP protocol see:
* [w3.org](https://www.w3.org/Protocols/HTTP/1.1/rfc2616bis/draft-lafon-rfc2616bis-03.html)
* [iana.org](https://www.iana.org/assignments/message-headers/message-headers.xml#perm-headers)

### Useful Resources

Below are several useful links that have been used in testing:
* [https://reqbin.com/](https://reqbin.com/) for REST testing
* CURL has been used extensively to check the requests. Use `-v` for verbose logging - to see all header details
* In addition, [Socket Test](https://sourceforge.net/projects/sockettest/) was used to confirm the requests have been sent and to validate their structure. To do this, create a server instance with an IP on the same subnet as the PLCSIM Advanced and configure the connection to point to that IP and port. You can use PuTTY to confirm the server is listening
* [PLCSIM Advanced](https://support.industry.siemens.com/cs/document/109772889/trial-download%3A-simatic-s7-plcsim-advanced-v3-0?dti=0&lc=en-WW) was used to simulate the PLC and test TCP/IP traffic. It was installed on a Windows 7 VM.

The general structure of the messages between the server and the client can be broken down as follows:

### Request Structure

1. A header which specifies the method, connection string, authentication details, data formatiing, etc. In the case of the server, it will involve the response code (ok, error), response length, server details etc. An example of a client header:

```
GET /api/v2.0.0/status HTTP/1.1
Host: localhost
User-Agent: curl/7.47.0
accept: application/json
Authorization: Basic <AuthString>
Accept-Language: en_US
```

2. The body of the message. This is the data that will be stored or actioned by the client and server respectively. It ranges from parameters that need to be deleted to data values returned to the client. The format of this data is specified via the `accept` variable in the header. 

```
{
    "allowed_methods": null,
    "battery_percentage": 100.0,
    "battery_time_remaining": 13680,
    "distance_to_next_target": 0.0,
    "errors": [],
    "footprint": "[[0.506,-0.32],[0.506,0.32],[-0.454,0.32],[-0.454,-0.32]]"
}
```

Examples of different requests will be illustrated below. 

### GET

**Example 1.** A call to get status data from MiR 

```
curl -X GET "http://localhost/api/v2.0.0/status" -H "accept: application/json" -H "Authorization: Basic <AuthString>" -H "Accept-Language: en_US" -v
```

The actual request headers are sent as:

```
GET /api/v2.0.0/status HTTP/1.1
Host: localhost
User-Agent: curl/7.47.0
accept: application/json
Authorization: Basic <AuthString>
Accept-Language: en_US
```

Where `<AuthString>` is the authorization string. In our case it's a hashed `username:password` combination (mind that the colon is included). This has to be stored within the PLC as a string as it doesn't have any cryptographic functions. There is no content data. 

This is basically how all of the GET requests are made. The only thing you have to worry about is ensuring the receive buffer has enough space to store the response.

### POST

**Example 1.** A request to create a new software backup (no content data)

```
curl -X POST "http://192.168.0.124/api/v2.0.0/software/backups" -H "accept: application/json" -H "Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==" -H "Accept-Language: en_US" -v
```

This POST request doesn't require any parameters. See Example 2 for feeding parameters.

**Header**

```
POST /api/v2.0.0/software/backups HTTP/1.1
Host: 192.168.0.124
User-Agent: curl/7.55.1
accept: application/json
Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==
Accept-Language: en_US
```

There is no data to post. You should get HTTP response 201

**Example 2.** A request to generate a new error report (with content data)

```
curl -X POST "http://192.168.0.124/api/v2.0.0/log/error_reports" -H "accept: application/json" -H "Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==" -H "Accept-Language: en_US" -H "Content-Type: application/json" -d "{ \"description\": \"string\"}" -v
```

This POST request does require submitting data. In the curl command this is specified after the '-d' command. 

Take note that this data needs to be submitted in JSON format. Indentation does not need to be preserved but letter case does. So, `"Description" != "description"`.

**Full Raw Request**

```
POST /api/v2.0.0/log/error_reports HTTP/1.1
Host: 192.168.0.124
User-Agent: curl/7.55.1
accept: application/json
Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==
Accept-Language: en_US
Content-Type: application/json
Content-Length: 24

{"description": "Hello"}
```

In terms of the PLC, make sure that the internal receive buffers are big enough for the content. If you're getting any errors it's probably on them.

### PUT

**Example 1.** Issue a new value and label to a register

```
curl -X PUT "http://192.168.0.124/api/v2.0.0/registers/1" -H "accept: application/json" -H "Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==" -H "Accept-Language: en_US" -H "Content-Type: application/json" -d "{ \"value\": 2, \"label\": \"miko\"}" -v
```

Here, we've actually used the hashed username and password. You'll have noticed that the data is escaped, i.e: `\"value\"` instead of `"value"`. This is unnecessary in a PLC; the data can be entered as `{"value": 2, "label": "miko"}`. 

This is basically just like a POST request. Same rules for data formatting apply: keep the content data in JSON format, letter case is important and you don't need to escape any characters or worry about indendation.

**Header**

```
PUT /api/v2.0.0/registers/1 HTTP/1.1
Host: 192.168.0.124
User-Agent: curl/7.55.1
accept: application/json
Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==
Accept-Language: en_US
Content-Type: application/json
Content-Length: 30
```

**Data**

```
{
  "value": 2,
  "label": "miko"
}
```

**Raw Header & Data Combined**

```
PUT /api/v2.0.0/registers/1 HTTP/1.1
Host: 192.168.0.241
User-Agent: curl/7.55.1
accept: application/json
Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==
Accept-Language: en_US
Content-Type: application/json
Content-Length: 30

{ "value": 2, "label": "miko"}
```

### DELETE

**Example 1.** Delete a specific error log

```
curl -X DELETE "http://192.168.0.124/api/v2.0.0/log/error_reports/14" -H "accept: application/json" -H "Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==" -H "Accept-Language: en_US" -v
```

**Header**

```
DELETE /api/v2.0.0/log/error_reports/15 HTTP/1.1
Host: 192.168.0.124
User-Agent: curl/7.55.1
accept: application/json
Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==
Accept-Language: en_US
```

Successfull HTTP response has code 204.

This is basically just like a GET only you change the method at the very begining... should really be one function. The only difference is that it does not return any content data even when the request has been successful. This is important as no content could be interpreted as an error so changes to the code have been made to avoid that problem.

## JSON Handling

At the moment the library does not provide any helper functions for encoding and decoding JSON data. 

## REST Library

The REST library contain four function blocks that realize REST requests, two helper functions from the LHTTP library and two data blocks that can be used as templates.

| Name          | Type | Description                                    |
|---------------|:----:|------------------------------------------------|
| REST_Get      |  FB  | Implements a REST Get request                  |
| REST_Post     |  FB  | Implements a REST Post request                 |
| REST_Put      |  FB  | Implement a REST Put request                   |
| REST_Delete   |  FB  | Implements a REST Delete request               |
| REST_Param    |  DB  | Contains parameter data for function blocks    |
| REST_Response |  DB  | Contains the response data from a REST request |
| LHTTP_FindStringInArray |  FB  | Searches an array of chars for a given string |
| LHTTP_ExtractString FromArray |  FB  | Extracts a string between two specified text parts from an array of chars |

All four of the FB that implement REST API have following parameters:

| Name         | Declaration |     Type     | Description                                                                               |
|--------------|:-----------:|:------------:|-------------------------------------------------------------------------------------------|
| execute      |    Input    |     Bool     | Send the REST request                                                                     |
| conn         |    Input    |  TCON_IP_V4  | Specifies the connection details. The IP must match the IP in the url                     |
| url          |    Input    |    String    | The URL of the request. Must contain IP address of the client                             |
| data         |    Input    |    String    | Additional data to be submitted, if needed. Data must be in a JSON format to be accepted  |
| responseData |    InOut    | Array[0...n] | Data received from the client. Array must start at 0                                      |
| done         |    Output   |     Bool     | Job finished                                                                              |
| busy         |    Output   |     Bool     | Job is being processed                                                                    |
| error        |    Output   |     Bool     | An error has occurred in the processing of the FB                                         |
| statusID     |    Output   |     USInt    | Specifies the source of the internal error. See LHTTP documentation for details           |
| status       |    Output   |     Word     | Internal status/error code. See LHTTP documentation for details                           |
| responseCode |    Output   |     UInt     | HTTP/REST response code (200 - success, > 400 - error, etc)                               |
| length       |    Output   |     UDInt    | Length of the received data                                                               |

## Integration

The steps to integrate the library into your program are straightforward. 

1. Open the library in the TIA portal
1. In the libraries tab, open the `REST v01` library. You'll find the function blocks in the Types section and default Data Blocks in the Master copies section. Copy the blocks needed for your project.
1. It's a good idea to use the data blocks as a starting point. See below for how a typical FB would look like, using the provided DBs:

Insert REST_Get_Block_cropped.png

Important notes:

* Make sure the connection parameter is configured with the correct hardware and the IP address matches the one in the url
* If you're using more than one block, make sure to change the connection ID in the connection parameter

## Extra Documentation

It's really well documented and the installation + implementation details are straightforward. See [this link](https://support.industry.siemens.com/cs/document/109763879/library-for-http-communication-(lhttp)?dti=0&lc=en-US) for more details. If not available, see [the link in Documentation](<Documentation/LHTTP Original Documentation.pdf>).
