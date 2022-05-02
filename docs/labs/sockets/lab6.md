# **Lab 6 - Socket Basics in Python**

![echo server picture](https://1.bp.blogspot.com/-QRugwxV8vb0/WHbfmHTu7HI/AAAAAAAADug/V1pYGpEzTpQvcfaoVh8qZN5d2xyERp4HACLcB/s1600/echoImage.png)

[Lab 6 Assignment page on Canvas](https://canvas.uw.edu/courses/1373089/assignments/5369622)

## Overview

In the remaining weeks of the quarter, we will create a network service using the Python sockets API. In this lab, we will explore the basic use of the sockets API along with some related topics about text and data representation in network protocols.

If you need help with getting started with Python, Jarett has created a video tutorial with timestamps for each section here: <https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=685386aa-3431-4e2a-a46a-ab600050535b>


## Instructions

Accept the [Github Classroom assignment](https://classroom.github.com/a/UQQt38l5) and clone the base repository for the assignment using `git clone <URL>`.

* This repository contains a markdown template for your lab report and starter code for the projects we will work on during the remaining weeks of the course.
* Create a new branch called `lab6-echo-server` with `git checkout -b lab6-echo-server`.

Review the first few sections of **[Real Python: Sockets Programming in Python](https://realpython.com/python-sockets/)** (through `Echo Client and Server`).

* Complete the questions in the report section below, referencing additional materials as needed. The report template is in the Github repository.

Complete the programming tasks:

* Use the provided code for **echo_server.py** and **echo_client.py** to complete the following tasks:
* Create a server based on the RealPython **echo server** that implements the four modes of operation described below.
	1. Echo input back to client
	2. Send a personalized welcome back to the client, e.g., "world" => "Hello, world!" (1:15:21 in the Python Tutorial video may help with command-line arguments)
	3. Parse the first line of an arbitrary HTTP request into a Python dictionary (16:39 in the Python Tutorial video may help with dictionaries)
	4. Parse a single HTTP header into a Python dictionary
* Create a client based on the RealPython **echo client** and use it to test your server implementation. 
* Test your code to ensure that all parts work as intended.

## Report

*Questions 1 - 5 are focused on the Python Sockets library and intended to help you better understand the sample code you are building from. Pay close attention to the reading and cite additional resources as needed.*

1. What two values are returned by the accept() function in the Python socket library?

2. Briefly describe the difference between bind(), listen(), and accept()?

3. How can we get a server socket to listen and accept connections on all available IPv4 interfaces and addresses?

4. How do we create a server socket that restricts communication to processes running on the same host?

5. What happens if a client or server calls recv() on a connection, but there isn't any data waiting to be processed?

*Questions 6 - 10 are focused on character encoding as discussed in the lab. Cite additional resources as needed.*

6. How many characters are in the basic ASCII character set? How many bits are required to represent an ASCII character value?

7. How many bytes are used to encode an ASCII value in UTF-8?

8. What is the UTF-8 encoding of your favorite emoji (provide the answer in hex)?

9. Which character encoding is specified by the RFCs for HTTP/1.0 and HTTP/1.1?

10. What line ending is used to delimit request and response headers in HTTP/1.0 and HTTP/1.1?

## Resources

* **[Real Python: Sockets Programming in Python](https://realpython.com/python-sockets/)**
* **[Strings, Unicode, and Bytes in Python 3](https://medium.com/better-programming/strings-unicode-and-bytes-in-python-3-everything-you-always-wanted-to-know-27dc02ff2686)**
* **[Real Python: Python Encodings Guide](https://realpython.com/python-encodings-guide/)**
* **[Line Endings (Wikipedia)](https://en.wikipedia.org/wiki/Newline)**
* **[CRLF (Mozilla Developer Glossary](https://developer.mozilla.org/en-US/docs/Glossary/CRLF)**
* **[How HTTP Works under the Hood](https://drstearns.github.io/tutorials/http/)**
* **[Hypertext Transfer Protocol -- HTTP/1.0](https://tools.ietf.org/html/rfc1945)**
* **[Hypertext Transfer Protocol -- HTTP/1.1](https://tools.ietf.org/html/rfc2616)**