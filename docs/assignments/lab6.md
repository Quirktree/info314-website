# **Lab 6 - Socket Basics in Python**

![echo server picture](https://1.bp.blogspot.com/-QRugwxV8vb0/WHbfmHTu7HI/AAAAAAAADug/V1pYGpEzTpQvcfaoVh8qZN5d2xyERp4HACLcB/s1600/echoImage.png)

[Lab 6 Assignment page on Canvas](https://canvas.uw.edu/courses/1373089/assignments/5369622)

## Overview

In the remaining weeks of the quarter, we will create a network service using the Python sockets API. In this lab, we will introduce the basic concepts of network programming with a Sockets API and build a pair of toy applications.

## Instructions

* Accept the Github Classroom assignment ADD GITHUB CLASSROOM LINK HERE and clone the base repository for the assignment using `git clone <URL>`
* Review the first few sections of [Real Python: Sockets Programming in Python](https://realpython.com/python-sockets/) (through `Echo Client and Server`).
* Create a new branch called `lab6-echo-server` with `git checkout -b lab6-echo-server`. Within your new branch:
	* Implement the basic **echo client** and **echo server** presented in the RealPython guide.
	* Modify the echo client to allow you to include a custom message from the command line, e.g., `echo-client “yay python”`. (Hint: Use `sys.argv`)[^arguments]
	* Encode[^encoding] the message as `UTF-8` text before sending (see linked resources).
	* Test your code and ensure that the applications work as intended.

[^encoding]: [https://www.pythoncentral.io/encoding-and-decoding-strings-in-python-3-x/](https://www.pythoncentral.io/encoding-and-decoding-strings-in-python-3-x/)
[^arguments]: [https://stackabuse.com/command-line-arguments-in-python/](https://stackabuse.com/command-line-arguments-in-python/) 