# GET Function Block Changes

So, to begin with I'm not going to mess with most of the code since it handles the HTTP requests just fine. The goal here is to amend the request calls
as to be lined up with the RESTful standard, add an authentication string, etc.

# MiR GET Request

Using curl, a standard get request looks like:

```
curl -X GET "http://192.168.1.173/api/v2.0.0/status" -H "accept: application/json" -H "Authorization: Basic YWRtaW46OGM2OTc2ZTViNTQxMDQxNWJkZTkwOGJkNGRlZTE1ZGZiMTY3YTljODczZmM0YmI4YTgxZjZmMmFiNDQ4YTkxOA==" -H "Accept-Language: en_US"
```

We can use the `-v` extension to get the verbose request and see what the headers actually are. It returns:
```
GET /api/v2.0.0/status HTTP/1.1
Host: localhost
User-Agent: curl/7.47.0
accept: application/json
Authorization: Basic <AuthString>
Accept-Language: en_US
```

This is indeed born out by their documentation.

# Siemens LHTTP GET Request

Now,  we can have a look at the current Siemens headers. There's a whole bunch of code inside of the FB that serves auxilarry function: timers, connections, eRrors, etc. 
We're not really concerned with those as that functionality remains unchanged. 

Fundamentally, the only thing that'll be different is the HTTP request header and the 
data returned by the server. Instead of being in an text/html format it'll be in the  json format.

The Siemens GET Header is:

```
GET /$base_path/$tempQuery HTTP/1.1
Host: %host
User-Agent: SIMATIC S7-1500
accept: text/html
Accept-Encoding: identity
Accept-Charset: utf-8
```

Based on the following code:

```
// Put together GET request
// Strg_TO_Chars conversion is done several times to utilize maximum length of parameters url and data
Strg_TO_Chars(Strg := CONCAT(IN1 := 'GET ', IN2 := #tempPath), // Path cannot be longer than 249 characters --> save to use CONCAT
			pChars := 0,
			Cnt => #tempLen,
			Chars := #statRequest.chars);
#tempPos := UINT_TO_INT(#tempLen);

Strg_TO_Chars(Strg := #tempQuery,
			pChars := #tempPos,
			Cnt => #tempLen,
			Chars := #statRequest.chars);
#tempPos += UINT_TO_INT(#tempLen);

Strg_TO_Chars(Strg := CONCAT(IN1 := ' HTTP/1.1$R$LHost: ', IN2 := #tempHost), // Host will not use all 254 characters --> save to use CONCAT
			pChars := #tempPos,
			Cnt => #tempLen,
			Chars := #statRequest.chars);
#tempPos += UINT_TO_INT(#tempLen);

Strg_TO_Chars(Strg := '$R$LUser-Agent: SIMATIC S7-1500 (LHTTP)$R$LAccept: text/html$R$LAccept-Encoding: identity$R$LAccept-Charset: utf-8$R$L$R$L',
			pChars := #tempPos,
			Cnt => #tempLen,
			Chars := #statRequest.chars);
#statRequest.length := INT_TO_UINT(#tempPos) + #tempLen;

#statSend.connect := true; // Connect
#statSend.req := true; // Send request
#statTimeoutIn := true; // Start timer
#statFBState := #STATE_SEND;
END_REGION
```

Note that the `$R$L` is an ASCII characters for CRLF (carriage return & line feed) 

It's clear that some variables should be hardcoded. Seing as this library is designed to communicate with a single MiR robot there's no need to let the user change the  response type
(we're only working with JSON). The authorization string will also be hardcoded in., as will the `Accept-Language` variable.  So really, the bulk of the work is done for us.



