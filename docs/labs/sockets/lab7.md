# Lab 7 - proxy part 1

![proxy image](https://hydrasky.com/wp-content/uploads/2017/10/proxy_diagram.png)

[TODO Lab 7 Assignment page on Canvas]()

## Python concepts

* program structure
* find(), split(), strip()
* slicing lists / strings
* try / except
* dictionary
* control characters, e.g., `\r\n`

## Instructions

### Getting Started 

As always, create a new branch dedicated to this lab within your http-proxy repository. All code development for this task should take place in `http-proxy.py`.

HTTP proxies incorporate server and client functionality into a single daemon. As a server, your proxy listens and process the requests it receives from browsers and other HTTP clients. As a client, your proxy will pass the clients' requests on to the intended destination and handle the responses in return. We can implement this behavior by creating both types of sockets within the same application and passing the relevant information between them.

In this task, we will work exclusively on the server functionality of the proxy. Specifically, we'll build a server that processes the incoming TCP stream and parses out HTTP GET requests. In future assignments, we'll extend our code to handle other message types and implement the client functionality.


### Main Loop (Event Loop)

To start out this lab, it will be helpful to reference the server logic you created in the `main` function of your echo server. On the server side, our proxy will build on connection and communication flow we established in that task. Namely, we want to provide a way for a web browser to repeatedly open new connections in order to send HTTP requests and receive the corresponding response (one request/response per connection).

Within the echo server, you wrote a loop that would receive the next connection from `accept()` and begin to process it. This loop is the basis for handling the connections that will be made from web browsers that are configured to speak through the proxy. You should be aware, however, that the connection handling process for a proxy is more complex than what we created for the echo server. 

To provide a minimally functional echo server, we could get away with making a single call to `Socket.recv()` per connection. This would accept the first chunk of data received and enable us to echo it back from the client. There are limitations on this approach since it doesn't take advantage of the _stream_-based nature of a TCP connection (or even fully recognize its constraints). The benefit to us, however, is that we didn't have to think to critically about how to recognize the end of an _echo request_.

#### Stream-Oriented Protocols and the Sockets API

As you move forward in the next few labs, you will need to wrestle with what it means to work with a stream-oriented protocol and the way in which that is implemented within the Sockets API. The main implication of the stream metaphor is that we don't have an easy way to recognize where messages (like HTTP requests and responses) start and end. 

The only thing we can take for granted when we call `Socket.recv()`, is that we received a new chunk of data from the stream. Depending on the behavior of the TCP client and server, the chunk of data could represent an entire message (as we assumed for our echo), part of a message, or even multiple messages. Likewise, we need to be aware that a call to `Socket.recv()` when we have already received everything from the other side will hang until a timeout occurs and the connection is closed.

Further discussion of this can be found in the latter sections of the [Real Python tutorial](https://realpython.com/python-sockets/#application-client-and-server)

In order to read from a stream, you will need to wrap `Socket.recv()` within an infinite loop, breaking out only when you can determine that you've received a complete message **-or-** when you detect that the connection is closed from the other side. In order to know where messages begin and end, we have to inspect the incoming bytes based on HTTP message syntax, which will be the primary goal of this lab.

**This is worth repeating one more time:**

!!! important
    Don't assume that `recv()` will return a complete HTTP message in one call. The Sockets API does not even guarantee that it will return the full number of bytes you requested, i.e., `Socket.recv(1024)` returns _up to 1024 bytes_ of data from the stream. 

#### Store Incomplete Messages in a Buffer

Since it is up to you to identify and reconstruct messages out of the stream, you should create a buffer that will hold your data while you work to identify complete HTTP messages. A buffer in this case is nothing more than a python bytes object that you append to each time a `recv()` returns successfully, for example:

```python
# Declare an empty buffer
buffer = b''

while True:
    data = conn.recv(1024)
    
    # break if the connection was closed
    if not data:
        break
    
    # add new data to the buffer.
    buffer = buffer + data

```

### Working HTTP Messages

In order to recognize HTTP messages, we will attempt to _parse_ our buffer each time we add new data in order to test whether we can read the message through to its proper end. When this operation succeeds, we will be able to move forward into the client portion of the proxy that we'll explore in upcoming labs. While this operation fails, we will continue to receive new data and add it to our buffer as discussed in the previous section.

Your main task for this lab is to build a simple HTTP parser that accepts a byte buffer and attempts to parse the most basic type of HTTP message, i.e., an HTTP/1.0 GET request (ignore other types of requests for now). In order to parse a GET request, you will read through the message a line at a time until you receive back-to-back CRLF (HTTP end-of-line) sequences indicating that you've reached the end of the HTTP headers. This task may require a small amount of research into [RFC-1945](https://tools.ietf.org/html/rfc1945) to review the HTTP/1.0 specification, but it builds directly off of the string and byte manipulation operations you practiced in the previous lab.

From a code standpoint, please encapsulate your parsing logic in a function called `parse_message`. This message should accept a buffer of bytes as input and return two values, i.e., a parsed message stored in a python dictionary and a byte buffer containing data that was left after parsing the message (which could be part of another request). If you are not able to parse the message completely, you can indicate this by returning `None` and the original byte buffer.

Based on this description, your parsing function should resemble:

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
HTTP messages follow a relatively simple syntax that can be used to make decisions about how to break the message into smaller parts (using tools like Python _slicing_ and the functions `find()`, `strip()`, and `split()`).  Assume that all messages are conformant with HTTP/1.0 and encoded as ISO-8859-1.

For GET requests, the message structure is based entirely around lines and the _Carriage Return / New Line_ line separators (think back to previous labs in which we sent GET requests using netcat). It's tempting for many students to read this and start parsing with a `data.split('\r\n')`. This strategy rarely works. 

Instead, we recommend that you create a dedicated function to split a single line off the buffer. You can do this with `find()`, `strip()`, and _slices_.

Before you go any further, make sure you have a basic understanding of HTTP/1.0 message structure. RFC-1945 is the ultimate authority for message structure, but I recommend checking out the other resources as well for a concise overview of the request/response format.

- **[RFC-1945](https://tools.ietf.org/html/rfc1945)**
- **[How HTTP Works under the Hood](https://drstearns.github.io/tutorials/http/)**
- **[HTTP syntax overview](/assignments/proxy-labs)**



### Output

After successfully parsing a request, you should print the following summary and close the connection (return to the start of your loop to listen for new requests).

**Request summary**

* **Connection Source:** \<IP address returned from the call to Socket.accept()\>
* **HTTP Method:** \<Name of method, e.g., GET, OPTION, or POST\>
* **Destination:** \<URI extracted from the request\>
* **Headers:** \<Comma delimited list of header names\>



!!! info "Producing a comma-delimited list of header names"
    If you're unfamiliar with python, you might find it challenging to produce the final part of this summary. The following method assumes you've stored your headers in a list, and that each header is parsed into a dictionary with a `name` and `value` as presented in the previous lab, i.e., `[{'name': 'Host', 'value': 'www.google.com'}, ...]`

    ```python
        header_names = [ h['name'] for h in headers ] # loop over all headers and save the name field in a list
        out = ', '.join(header_names) # join a list of strings with a comma
    ```

### Testing your code with `test.py`

The resources directory of the project repository contains a simple python script called test.py that you can use to send HTTP requests from a file into your proxy code.

```bash
# Send the request from sample-request.txt 
# one line at a time with a short delay between
python3 test.py 9999 sample-request.txt

# Send the request from sample-request.txt 250 bytes at a time 
# with a short delay between
python3 test.py 9999 sample-request.txt 250
```

### Capturing proxied requests for testing

A couple of sample requests are included in the directory, but the following method may be helpful if you would like to capture valid requests that you can use for testing:

-   Open ncat to listen for incoming connections, e.g., `ncat -o <FILENAME> -l <PORT>`
-   Configure Firefox with an HTTP proxy on `127.0.0.1 <PORT>`
-   Enter the URL of an HTTP-only site into the FF address bar (the request will hang)
-   Manually stop the request from the browser
-   Verify that the request was captured in ncat (ncat will close automatically)
