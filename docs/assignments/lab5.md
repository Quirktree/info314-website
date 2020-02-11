# **Lab 5 - Socket Basics in Python**

![echo server picture](https://1.bp.blogspot.com/-QRugwxV8vb0/WHbfmHTu7HI/AAAAAAAADug/V1pYGpEzTpQvcfaoVh8qZN5d2xyERp4HACLcB/s1600/echoImage.png)

## Overview
In the remaining weeks of the quarter, we will create a network service using the Python sockets API. In this lab, we will introduce the basic concepts of network programming with a Sockets API and build a pair of toy applications.

## Instructions

* Accept the Github Classroom assignment ([link](https://classroom.github.com/a/Q974X4tR)) and clone the base repository for the assignment.
* Review the first few sections of [Real Python: Sockets Programming in Python](https://realpython.com/python-sockets/) (through `Echo Client and Server`).
* Within a new branch called `lab5-echo-server`:
	* Implement the basic **echo client** and **echo server** presented in the RealPython guide.
	* Modify the echo client to allow you to include a custom message from the command line, e.g., `echo-client “yay python”`. (Hint: Use `sys.argv`)[^arguments]
	* Encode[^encoding] the message as `UTF-8` text before sending (see linked resources).
	* Test your code and ensure that the applications work as intended.
* Submit a working version of your code via pull request.

## Deliverables
* Open the lab report template (available in starter repository as lab5-report.md). In it, please answer the following questions:
	1. There are two major variants of Python (version 2 and 3). Briefly explain why it is important to know which version you have installed and are using to run your code.
	2. In your own words, explain the difference between server and client sockets. How are these differences seen in the sockets API?
* Complete the code and pull request as described above. Submit a link to your pull request via Canvas.

[^encoding]: [https://www.pythoncentral.io/encoding-and-decoding-strings-in-python-3-x/](https://www.pythoncentral.io/encoding-and-decoding-strings-in-python-3-x/)
[^arguments]: [https://stackabuse.com/command-line-arguments-in-python/](https://stackabuse.com/command-line-arguments-in-python/) 