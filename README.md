# S Rest API
A solution to consume and post REST API within Siemens S7 PLCs.

## Background

## Design

The LHTTP library has been used as a starting point for making HTTP requests and receiving data. It's been extended insofar as adding additional parameters for configuring the header details and adding function blocks to decode/encode simple parameters from/into the JSON format. Two additional methods, PUT and DELETE, have been also added. 

The Library was designed with straightforward calls in mind. It **does not** support:
* Particularly complex JSON formats such as arrays of objects, triple nested objects, etc.
* Authentication standards beyond basic (usually encrypted in an authentication string)

## API Design Breakdown

For a detailed breakdown of the code changes see the individual pages:
* [GET](REST API Requests/Get Design Changes.md)
* [PUSH](#)
* [PULL](#)
* [DELETE](#)

## LHTTP Library

It's really well documented and the installation + implementation details are straightforward. See [this link](https://support.industry.siemens.com/cs/document/109763879/library-for-http-communication-(lhttp)?dti=0&lc=en-US) for more details. If not available, see [the link in Documentation](Documentation/LHTTP Original Documentation.pdf).
