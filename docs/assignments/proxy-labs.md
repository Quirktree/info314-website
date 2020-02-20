# HTTP Proxy Labs 

## What is a proxy?

A proxy is a network service that receives a connection and reads data
from a client, acts on it, and passes that data out to another server.
The proxy will also read the server&apos;s response, act on it, and send it
back to the client. This simple pattern appears frequently in networks
and distributed systems and quite frequently provide

The value provided by a proxy is determined by the actions it takes on
the data. Here are a few applications of HTTP proxies:

-   security (e.g., inspecting HTTP content for malware)
-   policy enforcement (e.g., preventing employees from accessing unauthorized content at work)
-   software development (e.g., decrypting and examining HTTPS traffic)

For this sequence of assignments, we are building a proxy to log
metadata about HTTP requests and responses.

## Project Overview

This sequence of labs is divided into 3 parts.

1. Create a simple TCP server that is able to parse incoming HTTP/1.0 GET requests and generate a log of requests that have been received.
2. Extend your server so that it can parse the remaining HTTP/1.0 message types and build a new HTTP message from the fields it received.
3. 

## Background Material

### Application Layer Messages

#### Data structure

Application protocols follow predictable patterns to marshall more
complex data into byte-oriented message structures. The following
patterns are commonly used for marshalling/unmarshalling data and will
be discussed briefly in lab:

-   Delimiters
-   Type/Length/Value
-   Fixed length fields

#### Text encoding

In addition to the higher-level concern of marshalling the data
structures into a stream of bytes, network applications must also define
the manner in which they encode/decode between characters and byte
representation. There are many different ways to encode a set of
characters into bytes. These are generally grouped into single and
multi-byte encodings based on the number of bytes that are required to
encode each character.

The most prominent single-byte encodings are ASCII and ISO-8859-1, which
use 7-bits and 8-bits per character respectively. These encodings are
simple to work with, but they are quite limited in the size of alphabet
that they can support.

The advantage of multi-byte encodings, such as those defined by the
Unicode standard, is the ability to represent different languages,
alphabets, and sets of symbols within the application.

One of the most common encodings in modern applications is UTF-8, a
variable-length encoding that provides backwards compatibility with
ASCII while also supporting multi-byte encodings required for broad
Unicode compatibility.

For the bulk of our work on developing an HTTP proxy, we will utilize
the ISO-8859-1 and ASCII encodings (per the RFCs that define the
protocol).

#### Control Characters

As users, we're generally most concerned with the visible letters of the
alphabet -- or cute symbols such as 💩(represented as F0 9F 92 A9 in
UTF-8).

As developers, we often need to pay attention to the non-printable
character codes. For the sake of parsing the HTTP protocol, we will
concern ourselves with various types of white space, including the CRLF
(0D 0A in ASCII, ISO-8859-1, and UTF-8) sequence that is used to signify
the end of a line in HTTP messages.

### HTTP Message Structure

HTTP messages may be formatted as ASCII (per RFC 7230) or ISO-8859-1
(per historic RFCs). Decoding text as `iso-8859-1` will allow for the
maximum versatility.

RFC 7230 defines the structures used to represent HTTP/1.1 requests and
responses https://tools.ietf.org/html/rfc7230#section-3 .

```
    HTTP-message   = start-line
                      *( header-field CRLF )
                      CRLF
                      [ message-body ]
```

####Requests 

```
request-line - method, URI,  and protocol version
[header]*
empty line
[message body]

```

#### Responses

    status-line:
            protocol version, 
            a success or error code, 
            and textual reason phrase
    [header]*
    empty line
    [message body]
