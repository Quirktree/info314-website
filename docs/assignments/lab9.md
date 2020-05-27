# Lab 9 - proxy part III

[Lab 9 Assignment page on Canvas](https://canvas.uw.edu/courses/1373089/assignments/5369626)

In this lab, we'll wire together the remaining components of a proxy based on the functionality we have built to parse / reconstruct messages.

## Instructions

### Getting Started

As always, create a new branch dedicated to this lab within your http-proxy repository. All code development for this task should take place in `http-proxy.py`.

In the last lab, we completed three major tasks: a) extended our parse_message function to handle different types of HTTP requests, b) created a function to build a message based on the dictionary structure created by the parser, and c) opened a new client socket based for each request received by the proxy.

The main objectives of this lab will be to extend parsing functionality to handle HTTP responses as well as requests and to complete our end-to-end message flow.

!!! info "HTTP Version"
    Please confirm that you have followed the instructions in previous labs to downgrade the HTTP version to HTTP/1.0. Newer versions of HTTP have more complicated connection management, where as HTTP/1.0 connections are used for just one request/response.

---

### Parsing Responses
Structurally, there is almost no difference between an HTTP request with a message body and an HTTP response. In fact, the only difference from your parser's perspective is the order of fields in the first line of the message.[^http-syntax]

If we know the type of message we're parsing before we start, it's easy to extend your parse_message() function to handle HTTP requests and responses. For our needs, this is an easy requirement since the proxy will always know whether it's receiving a request vs. a response based on which socket is in use.

To build this functionality for yourself, add a `message_type` argument to your parser function. Modify the behavior of the function to parse the first line properly based on this argument, and include the `message_type` in the parsed message. Once you're confident that this is working, update your build_message function to check the type field and produce the appropriate bytes. 

There are many ways to handle a field like message_type. The most obvious, though inefficient, approach is to simply pass a string value such as `REQUEST` or `RESPONSE` and to test for the value when it is used. 

From a software engineering perspective, we prefer (but don't require) you to use a data type that is built for this purpose, i.e., the enumeration. Enumerations are data types that map a fixed set of values to numeric constants, and they're useful for any type where we need to compare to a predefined list such as the type of socket or the bit flags of a TCP header.

While they require a little more effort, enumerations provide advantages in terms of compile-time error detection and run-time efficiency (over string comparisons).

The code below demonstrates how to create and use a MessageType enumeration within your code. 

```python
# Required imports
from enum import Enum, auto

# Enumeration to represent message types 
class MessageType(Enum):
    REQUEST = auto() # constant values are auto-assigned
    RESPONSE = auto() # constant values are auto-assigned

# Use the is operator rather than == to test an enumeration …
def parse_message(data, message_type):
    ...
    message['type'] = message_type
    if message['type'] is MessageType.REQUEST:
        # parse request fields ...

def main():
    ...

    # parse an incoming request
    parse_message(buffer, MessageType.REQUEST)

```

[^http-syntax]: More information available at [HTTP syntax overview](/assignments/proxy-labs) 

### Connection Flow
With your parse/build code working correctly, your proxy is almost complete. Turn you attention to completing the connection flow between a browser and the final destination of a request. A summary of this message flow for a single request/response appears below.

-   Upon receipt of a valid request, parse the URI to determine the host and port for the destination server that's been requested by the client.
-   Within your event handling loop, create a client socket to the destination associated with the new request. The host and port are provided by the previous step.
-   Reconstruct the request using the `build_message` function from the previous lab and send it out over your new client socket.
-   Set up a new loop to listen on this socket until you have received the HTTP response coming back from the web server. As with the original HTTP request, you'll need to keep trying until you can parse the complete request. 
-   The browser is still waiting for a response to its request. Reconstruct the response and send it back to the browser over the original server socket.
-   Close the connection to the client and resume waiting for new connections.

### Testing

You may wish to start out using the test.py function for testing purposes in this lab, though its capabilities are limited. In particular, test.py cannot properly send a response in "line mode" (you have to provide an integer argument to tell it to break the message into fixed size chunks).

When you reach the point that your code returns a successful HTTP response to the test script, you should move on to testing with the browser.

Firefox is the easiest browser to use for testing since it provides standalone proxy configuration. Other common browsers rely on OS level configuration for proxies and tend to cause more issues for students.

-   Open Firefox's general preferences/settings page
-   Scroll to the bottom of the page and select _Network Settings_ -&gt;
_Settings_
-   Under _Configure Proxy Access to the Internet_, select _Manual
proxy configuration_
-   Set the _HTTP Proxy_ to `127.0.0.1` with _Port_ `9999`
(match the port you used to run your proxy)
-   Apply your settings and open a new tab to navigate to an HTTP-based site, e.g.,
    -   http://www.washington.edu/
    -   http://neverssl.com
    -   http://mit.edu

---

### Output

Upon completing this task successfully, your code should proxy a request to a server and return the response back to the client over the existing network connection. If you've done this successfully, the pages should load successfully in your browser when you view a site via unencrypted HTTP.

You should also print out a brief summary of each request and response received for troubleshooting purposes. The exact format does not matter -- we will refine the output in the next lab.

<!-- You should also print out a visual log of incoming connections, combining elements of the request and response as explained here. The format we will use for this log is consistent with the HTTP access logs seen in popular web servers like nginx and Apache. -->

<!-- #### Log Entry Format
Each line of the log contains the following entries separated by spaces:

-   Remote host (client IP address)
-   Timestamp of when your proxy received the HTTP request
    -   wrapped in square brackets
    -   ??? hint "how to get time stamp in Python"
        ```python
        import datetime
        now = datetime.datetime.now()
        now.strftime('%d/%b/%Y:%H:%M:%S')
        ```
-   The actual request-line received from the client
    
    -   wrapped in double quotes
-   The HTTP status code returned by the target server in its response
-   Content-Length of the HTTP response, i.e., the number of bytes in
the payload
    
    -   zero if the payload is not present
-   Referer header, or a dash if not used
    -   wrapped in double quotes
    -   replace with a single dash if Referer is not present
-   User-Agent header (contains information about the client's
browser/OS/etc)
    -   wrapped in double quotes
    -   replace with a single dash if the User-Agent is not present

<br> -->


<!-- **Examples**

```
127.0.0.1 [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326 "http://www.example.com/start.html" "Mozilla/4.08 [en] (Win98; I ;Nav)"
```

More information about the Apache access log is available at [scalyr detailed apache access log](https://www.scalyr.com/blog/detailed-introduction-apache-access-log/). -->
