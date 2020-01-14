# **Handshakes and TCP Streams**

[![](https://www.cloudflare.com/img/learning/cdn/tls-ssl/tcp-handshake-diagram.png)](https://www.cloudflare.com/img/learning/cdn/tls-ssl/tcp-handshake-diagram.png)


[*canvas link*](https://canvas.uw.edu/courses/1273846/assignments/4746317).

## Setup
---
For this lab, we are going to use the incredibly versatile [`ncat`](https://nmap.org/ncat/) utility to directly read and write data to and from HTTP(s) servers.

Some of you may already be familiar with a similar tool known as **netcat** (`nc` from the command line on most Linux or macOS machines). `ncat` is a re-implementation that adds some handy features (like SSL/TLS connections).

### Windows 
Please follow instructions to install `ncat` from https://nmap.org/ncat/. You can accomplish this either by installing the `nmap` port scanner suite or by installing the [portable executable](http://nmap.org/dist/ncat-portable-5.59BETA1.zip). 

Please test your installation by running `ncat` from a new Powershell console. 
!!! Note
    You may need to update your system path to include the directory in which you installed `ncat`.

### macOS and Linux
Install `nmap` from your preferred package manager (e.g., `apt` or `brew`) or follow instructions provided at [nmap.org](https://nmap.org/download.html).

## Capture
---
The instructions below can be used to manually perform an HTTP exchange with a server via the `ncat` tool. This process approximates the exchange that occurs between your web browser and a web server after you enter a URL into the address bar or click on a hyperlink.

!!! faq "Why don't I see any output"
    At this point, we're interacting directly with a server in a text-based protocol. The server will wait silently for you to enter a valid message. Per RFC 2616, the request you are sending is multiple lines and is terminated by two **new lines**. 
    
    If you wait too long or submit something that violates the specification, the server will respond with an error and close the connection.

### Capture Instructions

1. Create a [capture filter](https://wiki.wireshark.org/CaptureFilters) that will select traffic from TCP ports 80 and 443.
2. Launch a new Wireshark capture on your primary network interface with the capture filter from Step 1.
3. Launch a couple of new terminals or Powershell consoles and follow these steps to make HTTP requests to neverssl.com and wikipedia.org (in separate consoles):
    - **NeverSSL**
        - a. Open a new connection:
            - `ncat neverssl.com 80`
        - b. Start a new HTTP requests (terminated with **a newline**)
            - `GET / HTTP/1.1`
        - c. Add a **Host header** followed by  **two new lines**
            - `Host: neverssl.com`
    - **Wikipedia**
        - a. Open a new connection:
            - `ncat --ssl www.wikipedia.org 443`
        - b. Start a new HTTP requests (terminated with **a newline**)
            - `GET / HTTP/1.1`
        - c. Add a **Host header** followed by  **two new lines**
            - `Host: www.wikipedia.org`
4. Close your connections with _CTRL-C_
5. End your capture and save a copy to complete the remaining analysis.
6. Save the `ncat` ouput to a file (for deliverable 4).

## Analyze
---
### Plaintext (http)
Perform a search in wireshark (_CMD-F_ or _CTRL-F_) to find the _string_ `neverssl.com` in _packet details_.

!!! Note 
    You'll need to adjust the options for your search via dropdowns

This search _should_ lead you to the reassembled HTTP session. Review the summary of the reassembled TCP segments provided inside the _Packet Details_. For an alternate perspective, right click on any frame belonging to the connection and select _Follow -> TCP Stream_. Finally, right click on any frame in the connection and select _Prepare Conversation Filter -> TCP_

Answer the following questions (deliverable #5)

1. For the first 10 frames of the connection, identify:
    - Frame #
    - Source (_client_ or _server_)
    - Flags
    - Sequence number
    - Acknowledgement number
    - Length of payload (additional data)
1. Which information is used by the client to identify messages from the connection?
1. Which information is used by the server to identify messages from the connection?
1. Repeat the process with the last 5 frames of the connection.

### Encrypted (https)
Perform a new search to find `wikipedia.org`. Be careful with this search since the search term appears in the page content from `neverssl.com`. Once you have identified the a frame from this TCP connection, prepare a new conversation filter so that you can take a closer look at the session.

!!! Tip "Hint" 
    We are looking for the term to appear in a **Secure Sockets Layer** option. We won't actually see the contents of our HTTPS session since the connection is encrypted by the time we enter it.

!!! note
    It may be helpful to create a table or illustration that lists significant frames and their role in the session.

Answer the following questions (deliverable #6):

1. For the first 10 frames of the connection, identify:
    - Frame #
    - Source (_client_ or _server_)
    - Flags
    - Sequence number
    - Acknowledgement number
    - Length of payload (additional data)
1. Repeat the process with the last 5 frames of the connection.
1. Identify the first frame to contain a Server Sockets Layer header?
1. Looking at the first frames of the SSL/TLS session, identify the messages sent in the TLS handshake (hint: look for Handshake Type in the packet details).
1. Create a display filter to show the _server_ side of the conversation only.
1. Can you find any stream anomalies, e.g., duplicate data, out-of-order segments, or duplicate ACK (Hint: These are labeled in the Info field of the packet list)?


<br>
<br>

## Deliverables
---
Submit a lab report addressing each of the following requirements: 

!!! danger "Important"
    Include a copy of the packet capture and the full question text in your report.

1. Upload a copy of your packet capture.
1. Record the _capture filter_ you used in the first section of this lab.
1. Briefly describe the difference between a capture filter and a display filter. In what way do they serve different purposes.
1. Paste in the first 10 lines of each server's response from the `ncat` output.
1. Paste in questions/answers from the `neverssl.com` analysis.
1. Paste in questions/answers from the `wikipedia.org` analysis.
1. What function does the opening handshake of TCP accomplish?
1. Describe how you would identify stream anomalies by using sequence and acknowledgement numbers of TCP segments.




<br>
<br>




## Bonus
---
**(5 points)** Create a basic sequence diagram of the TCP segments sent in _wikipedia.org_ session.

??? danger "Do not diagram all 100+" 
    Do not diagram all 100+ frames of the wikipedia.org session, but you should include enough to illustrate the TLS handshake, the first few frames of encrypted data, and the closing of the TCP socket.

You can draw this out on paper and submit a scan or use any other tool capable of building diagrams. Take a look at https://websequencediagrams.com.

Your diagram should include annotations for the following details:

- Frame # (on each message)
- TCP Flags (on the setup/teardown handshake)
- TLS Handshake Protocol steps (if applicable)
- \# of bytes transferred (messages carrying data)

> Hint: Be careful to exclude reassembled messages from your diagram (e.g., frames labeled HTTP that represent multiple TCP segments).

<img style="height:400px;max-width:600px;" src="https://serverdensity-wpengine.netdna-ssl.com/wp-content/uploads/2017/02/Blog_TCP_IP_V2.png" />

