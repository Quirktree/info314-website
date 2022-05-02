# Lab 10 - proxy part IV

[Lab 10 Assignment page on Canvas](https://canvas.uw.edu/courses/1373089/assignments/5467235)

In this lab, we'll wire together the remaining components of a proxy based on the functionality we have built to parse / reconstruct messages.

## Getting Started

Create a new branch called `final-proxy-submission` dedicated to this lab within your http-proxy repository. All code development for this task should take place in `http-proxy.py`.

In the last lab, we extended the parse_message function to recognize HTTP response messages and completed the second half of the request/response forwarding connection flow. 

!!! info "HTTP Version"
    Please confirm that you have followed the instructions in previous labs to downgrade the HTTP version to HTTP/1.0. Newer versions of HTTP have more complicated connection management, where as HTTP/1.0 connections are used for just one request/response.

Our goal in this last lab is to complete any debugging and cleanup of the code and to replace the current printed output with a one-line summary from each connection.

## HTTP Headers Revisited
We have previously implemented an oversimplified version of the HTTP header, and on occasion, we may encounter a header that our proxy will be unable to parse due to the concept of continuation lines in RFC 1945. 

> HTTP/1.0 headers may be folded onto multiple lines if each continuation line begins with a space or horizontal tab. 

Extend your header parsing logic so that it can recognize a continuation line (prefixed with one or more space/tab characters) and append it's contents to the value section of the most recently parsed header.

## Testing and Debugging
Test your proxy thoroughly within the browser (Firefox recommended for this task). You should be able to visit a variety of HTTP-based (non-encrypted) websites and see that pages can load without causing any unexpected exceptions within the proxy. Academic institutions tend to be good candidates for this task since they are much more likely to work over HTTP without redirecting to a secure version of the site.

If you encounter any crashes or sites that will not load, attempt to debug the issue. Quite often, this is an indication that you forgot to handle the necessary edge cases in the HTTP message format, such as zero-length responses.

## Standardize Output
In previous labs, we focused more on the functionality of the proxy rather than the output. Before submitting your final work, clean up your existing output and print a single-line summary of each connection as described here. The format we will use for this log is consistent with the HTTP access logs seen in popular web servers like nginx and Apache. 

### Log Entry Format
Each line of the log contains the following entries separated by spaces:

-   Requester host (client IP address)
-   Timestamp of when your proxy received the HTTP request
    -   wrapped in square brackets
    -   ??? hint "Hint: how to get time stamp in Python"
        ```python
        import datetime
        now = datetime.datetime.now()
        now.strftime('%d/%b/%Y:%H:%M:%S')
        ```
-   The actual request-line received from the client (wrapped in double quotes)
-   The HTTP status code returned by the target server in its response
-   Content-Length of the HTTP response, i.e., the number of bytes in
the payload
    -   zero if the payload is not present
-   Referer header
    -   wrap the header in double quotes
    -   replace with a single dash (no quotes) if Referer is not present
-   User-Agent header (contains information about the client's
browser/OS/etc)
    -   wrap the header in double quotes
    -   replace with a single dash (no quotes) if the User-Agent is not present
<br> 


**Examples**

```
127.0.0.1 [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326 "http://www.example.com/start.html" "Mozilla/4.08 [en] (Win98; I ;Nav)"
```

More information about the Apache access log is available at [scalyr detailed apache access log](https://www.scalyr.com/blog/detailed-introduction-apache-access-log/).
