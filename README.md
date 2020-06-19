# S Rest API
A solution to consume and post REST API within Siemens S7 PLCs.

## Background

REST and HTTP...

Siemens S7 stuff...

## Design

The LHTTP library has been used as a starting point for making HTTP requests and receiving data. It's been extended insofar as adding additional parameters for configuring the header details and adding function blocks to decode/encode simple parameters from/into the JSON format. Two additional methods, PUT and DELETE, have been also added. 

The Library was designed with straightforward calls in mind. It **does not** support:
* Particularly complex JSON formats such as arrays of objects, triple nested objects, etc.
* Authentication standards beyond basic (usually hashed in an authentication string)

## API Design Breakdown

In short, the comms were left largely untouched. From a functional perspective there's no difference between an HTTP and a REST request beyond standard conventions. In fact, REST communicates via http protocols so in a sense it's a simple extension of what's already implemented.

For a detailed breakdown of the HTTP protocol see:
* [w3.org](https://www.w3.org/Protocols/HTTP/1.1/rfc2616bis/draft-lafon-rfc2616bis-03.html)
* [iana.org](https://www.iana.org/assignments/message-headers/message-headers.xml#perm-headers)

The general structure of the messages between the server and the client can be broken down as follows:

1. A header which specifies the method, connection string, authentication details, data formatiing, etc. In the case of the server, it will involve the response code (ok, error), response length, server details etc. An example of a client header:

```
GET /api/v2.0.0/status HTTP/1.1
Host: localhost
User-Agent: curl/7.47.0
accept: application/json
Authorization: Basic <AuthString>
Accept-Language: en_US
```

2. The body of the message. This is the data that will be stored or actioned by the client and server respectively. It ranges from parameters that need to be deleted to data values returned to the client. The format of this data is specified via the standard specified by the header. 

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

For a detailed breakdown of the code changes see the individual pages:
* [GET](<REST API Requests/Get Design Changes.md>)
* [PUSH](#)
* [PULL](#)
* [DELETE](#)

## JSON Handling



## LHTTP Library

It's really well documented and the installation + implementation details are straightforward. See [this link](https://support.industry.siemens.com/cs/document/109763879/library-for-http-communication-(lhttp)?dti=0&lc=en-US) for more details. If not available, see [the link in Documentation](<Documentation/LHTTP Original Documentation.pdf>).
