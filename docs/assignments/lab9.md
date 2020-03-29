# Lab 9 - proxy part III

[Lab 9 Assignment page on Canvas](https://canvas.uw.edu/courses/1373089/assignments/5369626)

In this lab, we'll wire together the remaining components of a proxy based on the functionality we have built to parse / reconstruct messages.

## Instructions

### Getting Started

As always, create a new branch dedicated to this lab within your http-proxy repository, and at the root of your repository, open up the `http-proxy.py` script you have been working on in Lab 6 - 7. You will continue working with the same file for Lab 8.

In the last lab, we completed the parsing functionality needed by our proxy and added a function to reconstruct the HTTP message with a few changes &emdash; such as downgrading the connection to `HTTP/1.0`. 


!!! info "HTTP Version"
    Please confirm that you have followed the instructions in previous labs to downgrade the HTTP version to HTTP/1.0. Newer versions of HTTP have more complicated connection management, where as HTTP/1.0 connections are used for just one request/response.

In this lab, we'll return our focus to the sockets in the main event loop in order to establish the remaining connections needed to proxy requests to the intended destinations and return responses to the client. 

---
### Server Port
To streamline grading for this final task, please modify your current script to accept a port from the command-line, e.g., `python3 http-proxy.py 9999`. This argument should override any constant value that you've been using until now for the server's port.

---
### Connection Flow
The overall connection flow is described below. Additional helper code is provided later in this section:

-   Upon receipt of a valid request, parse the URI _(code provided)_ to determine the host and port for the destination server that's been requested by the client.
-   Within your event handling code, create a new TCP connection to the new host and port. _This is a client connection and will be established in the same manner as the connection in your echo client of Lab 5.
-   Reconstruct the request using the `build_message` function from the previous lab and send it out over your new server-facing socket.
-   Set up a new loop to listen on this socket until you have receivedthe HTTP response coming back from the web server _(You'll need to keep trying until you can parse the complete request)_. 
-   Reconstruct the response and send it back to the client over the client-facing connection.
-   Close the connection to the client and resume waiting for new connections.

---
### Parsing the URI

In order to create a socket to a destination based on a URI, we need to
parse out several components and determine the hostname/address and port
that the client to the proxy had originally requested. The image below provides a good example demonstrating the different between a URL and URI.

![URI example](https://prateekvjoshi.files.wordpress.com/2014/02/uri-vs-url-vs-urn.jpg)

Since Python provides a URL parsing module, this is typically an easy task. In practice, however, there are quite a few edge cases. In order to constrain the difficulty of this lab, we are providing you with the
following function to parse out the host and port.

```python
# Be sure to add the import to the top of your code
from urllib.parse import urlparse

# returns the host and port
# run by doing:  h, p = parse_uri(dest)
def parse_uri(uri):
    uri = urlparse(uri)
    scheme = uri.scheme
    host = uri.hostname

    # urlparse can't deal with partial URI's that don't include the 
    # protocol, e.g., push.services.mozilla.com:443

    if host: # correctly parsed
        if uri.port:
            port = uri.port
        else:
            port = socket.getservbyname(scheme)
    else: # incorrectly parsed
        uri = uri.path.split(':')
        host = uri[0]
        if len(uri) > 1:
            port = int(uri[1])
        else:
            port = 80

    return host, port
```

---

### Testing

While building the remaining functionality, the simplest mode of testing is to leverage the `test.py` script provided in your start repository
(see instructions in Lab 7 and 8 for more details). When you reach the point that your code returns a successful HTTP response to the test
script, you should move on to testing with the browser.

Since it provides standalone proxy configuration, Firefox is the easiest browser to use for testing. Other common browsers rely on OS level configuration for proxies and tend to cause more issues for students.

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

You should also print out a visual log of incoming connections, combining elements of the request and response as explained here. The format we will use for this log is consistent with the HTTP access logs
seen in popular web servers like nginx and Apache.

#### Log Entry Format
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

<br>


**Examples**

```
127.0.0.1 [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326 "http://www.example.com/start.html" "Mozilla/4.08 [en] (Win98; I ;Nav)"
```

More information about the Apache access log is available at [scalyr detailed apache access log](https://www.scalyr.com/blog/detailed-introduction-apache-access-log/).
