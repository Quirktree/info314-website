# Lab 6 - proxy part 1

![proxy image](https://hydrasky.com/wp-content/uploads/2017/10/proxy_diagram.png)

## Python concepts

* program structure
* find(), split(), strip()
* slicing lists / strings
* try / except
* dictionary
* control characters, e.g., `\r\n`

## Instructions

### Getting Started 

As always, create a new branch dedicated to this lab within your http-proxy repository, and at the root of your repository, add a new script named `http-proxy.py` based on `resources/template.py`. This file will be the main deliverable for your project.

HTTP proxies incorporate server and client functionality into a single daemon. As a server, your proxy listens and process the requests it receives from browsers and other HTTP clients. As a client, your proxy will pass the clients' requests on to the intended destination and handle the responses in return.

In this task, we will work exclusively on the server functionality of the proxy. Specifically, we'll build a server that processes the incoming TCP stream and parses out HTTP GET requests. In future assignments, we'll extend our code to handle other message types and implement the client functionality.


### Main Loop (Event Loop)

Use a while loop to repeatedly `accept()` and process incoming requests. For each active connection, you will be making calls to `Socket.recv()` in order to fetch the next chunk of data from the TCP stream and process it.

!!! info "What it means to work with streams"
    Remember that TCP is a stream-oriented protocol and that the Sockets API is built around the metaphor of the stream of bytes. Each attempt to read data from a socket pulls data from the stream in small chunks, e.g., `Socket.recv(1024)` returns _up to 1024 bytes_ of data from the stream. 

    Don't assume that `recv()` will return a complete HTTP message in one call. The Sockets API does not even guarantee that it will return the full number of bytes you requested. In order to know where messages begin and end, we have to inspect the incoming bytes based on HTTP message syntax.

Since it is up to you to identify and reconstruct messages out of the stream, you should create a buffer that will hold your data while you work to identify complete HTTP messages. A buffer in this case is nothing more than a python bytes object that you append to each time a `recv()` returns successfully, for example:

```python
# Declare an empty buffer
buffer = b''

# Receive new data and add it to the buffer
data = conn.recv(1024)
buffer = buffer + data
```

### Getting to HTTP Messages

In order to recognize HTTP messages, you need to parse the bytes you've received and determine whether you have a complete message. Your task is to build a simple HTTP parser that accepts a byte buffer and attempts to parse an HTTP GET request (ignore other request types for now). We recommend that your parser returns the parsed message as a dictionary (see below) along with the bytes that were still left over in the buffer.

As noted, the data coming from the socket will be in bytestring format, meaning it will appear as `b'data'`  and not `'data'` as with a normal string. You cannot use string methods such as `split()` on a btyestring. Therefore you will need to decode the btyestring with the `str()` or `decode()` method to turn it into a string.

HTTP messages may be formatted as ASCII (per RFC 7230) or ISO-8859-1 (per historic RFCs). 

Decoding text as `iso-8859-1` will allow for the maximum versatility.

Based on this description, we can create a python function that resembles the following:

```python

def parse_message(data)
    # initialize an empty dictionary
    message = {}

    # parse bytes and assign key / value pairs such as ...
    message['method'] = # HTTP method name such as GET
    message['uri'] = # address or resource name from the request, such as www.uw.edu
    message['version'] = # HTTP version such as HTTP/1.0
    message['headers'] = [] # List of headers

    # If parsing is successful, return a completed message (if applicable) and unused bytes
    return message, unparsed_data

    # If parsing fails, return the entire buffer and an indicator that parsing was incomplete
    return None, data
```

#### Parsing
HTTP messages follow a relatively simple syntax that can be used to make decisions about how to break the message into smaller parts (using tools like Python _slicing_ and the functions `find()`, `strip()`, and `split()`). 

For GET requests, the message structure is based entirely around lines and the _Carriage Return / New Line_ line separators (think back to previous labs in which we sent GET requests using netcat). It's tempting for many students to read this and start parsing with a `data.split('\r\n')`. This strategy rarely works. 

Instead, we recommend that you create a dedicated function to split a single line off the buffer. You can do this with `find()`, `strip()`, and _slices_.

Before you go any further, make sure you have a basic understanding of HTTP/1.0 message structure. See [HTTP syntax overview](/assignments/proxy-labs) or [RFC 1945](https://tools.ietf.org/html/rfc1945).

### Output

After successfully parsing a request, you should print the following summary and close the connection (return to the start of your loop to listen for new requests).

**Request summary**

* **Connection Source:** \<IP address returned from the call to Socket.accept()\>
* **HTTP Method:** \<Name of method, e.g., GET, OPTION, or POST\>
* **Destination:** \<URI extracted from the request\>
* **Headers:** \<Comma delimited list of header names\>  

### Testing your code with `test.py`

The resources directory of the project repository contains a simple python script called test.py that you can use to send HTTP requests from a file into your proxy code.

``` bash
# Send the request from sample-request.txt 
# one line at a time with a short delay between
python3 test.py 9999 sample-request.txt

# Send the request from sample-request.txt 250 bytes at a time 
# with a short delay between
python3 test.py 9999 sample-request.txt 250
```

### Capturing proxied requests for testing

Use the following method to capture valid requests that you can use for testing:

-   Open ncat to listen for incoming connections, e.g., `ncat -o <FILENAME> -l <PORT>`
-   Configure Firefox with an HTTP proxy on `127.0.0.1 <PORT>`
-   Enter the URL of an HTTP-only site into the FF address bar (the request will hang)
-   Manually stop the request from the browser
-   Verify that the request was captured in ncat (ncat will close automatically)
