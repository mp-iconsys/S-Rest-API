# S Rest API
A solution to consume and post REST API within Siemens S7 PLCs.

## Contents

1. [Overview](#Overview)
1. [Rest API Breakdown](#REST-API-Breakdown)
1. [JSON Handling](#JSON-Handling)
1. [Installation](#Installation)
1. [Use Examples](#Use-Examples)
1. [Extra Documentation](#Extra-Documentation)

## Overview

The [LHTTP](https://support.industry.siemens.com/cs/document/109763879/library-for-http-communication-(lhttp)?dti=0&lc=en-WW) library has been used as a starting point for making HTTP requests and receiving data. It's been extended and tested to consume REST API. Two additional methods, PUT and DELETE, have been added and several features removed, to enable backwards-compatibility with S7-1200 PLCs. These were the SSL security protections and the DNS domain resolution.

The code has been designed and tested on SIMATIC S7-1500 CPU 1516-3 PN/DP, Article 6ES7516-3AN01-0AB0. It uses dynamic array sizes which are only supported for firmware **version >= 2.0** on S7-1500 and firmware **version >= 4.2** on S7-1200. 

The Library was designed with straightforward requests in mind. It **does not** support:
* Decoding or Encoding JSON content. The data is stored as a char array and no serialization is supported
* Authentication standards beyond basic. These have to be provided as a hashed username and password

## REST API Breakdown

In short, the comms were left largely untouched. From a functional perspective there's no difference between an HTTP and a REST request beyond standard conventions. In fact, REST communicates via http protocols so in a sense it's a simple extension of what's already implemented.

For a detailed breakdown of the HTTP protocol see:
* [w3.org](https://www.w3.org/Protocols/HTTP/1.1/rfc2616bis/draft-lafon-rfc2616bis-03.html)
* [iana.org](https://www.iana.org/assignments/message-headers/message-headers.xml#perm-headers)

### Useful Resources

Below are several useful links for REST testing:



The general structure of the messages between the server and the client can be broken down as follows:

### General Structure

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

### GET


### PUT


### POST

### DELETE


For a detailed breakdown of the code changes see the individual pages:
* [GET](<REST API Requests/Get Design Changes.md>)
* [PUSH](#)
* [PULL](#)
* [DELETE](#)

## JSON Handling


## Installation


## Use Examples


## Extra Documentation

It's really well documented and the installation + implementation details are straightforward. See [this link](https://support.industry.siemens.com/cs/document/109763879/library-for-http-communication-(lhttp)?dti=0&lc=en-US) for more details. If not available, see [the link in Documentation](<Documentation/LHTTP Original Documentation.pdf>).
