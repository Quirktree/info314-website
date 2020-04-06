# Lab 2 - Analyzing DHCP and ARP with Wireshark 
[Lab 2 Assignment page on Canvas](https://canvas.uw.edu/courses/1373089/assignments/5369617)

# Overview

The purpose of this lab is for students to get familiar with basic functionalities of Wireshark, and to be able to capture and analyze network traffic.

There will be multiple correct ways to complete this lab, we highly recommend using search engines to help you find solutions.

# Introduction to Wireshark

Wireshark is a tool designed to record and observe the messages that are sent over a network and to provide an analyst with tools to understand and troubleshoot the protocols associated with those messages.

## Starting a Capture

Open Wireshark and locate the Capture section of the launch screen. This section contains an input field for configuring capture filters and a list of physical and virtual interfaces that can be used to capture packets.

To the right of each interface, you should notice an animated “sparkline” which indicates the level of activity on the associated network. You can launch a capture on your active interface (likely Wi-Fi or Ethernet) by double-clicking on the respective label. 

If a capture filter was specified before you selected the interface, Wireshark ignores any packets that don't match the filter. If the filter input is empty, Wireshark records everything it sees on the wire. 

Recording continues until you press the stop icon in the top toolbar or close the application.

## Exploring a Capture

By default, Wireshark will divide capture-related content among three panes:

*   Packet List
*   Packet Details
*   Packet Bytes

The *Packet List* displays all packets in the current capture, summarizing important details about traffic flow and the type of data contained in the packet. 

The *Packet Details* view initially displays a layered summary of the protocols represented in the selected packet. A more detailed view of each protocol can be explored by clicking the arrow icons to the left of the summary. The order of the protocols in this view corresponds to the order in which each protocol’s header appears in the data frame. 

The *Packet Bytes pane* can be useful for understanding the basic structure of raw network messages. As you navigate through the details in this view, Wireshark will highlight the corresponding bytes in the Packet Bytes pane.

## Narrowing your Search

If you have any software, particularly a web browser, running in the background, you'll notice that Wireshark can quickly accumulate a large list of packets. Fortunately, the software provides a variety of tools to help you more easily locate packets relevant to your search.

The packet list can be modified non-destructively by adding/removing *display filters*[^filters] in the input field below the top toolbar. Display filters are a valuable analytic tool that allows you to remove noise and quickly focus on packets that match certain criteria.

In addition to writing your own display filters, you can access a variety of context-specific analysis tools by right clicking on any row of the Packet List or a field within the Packet Details pane. These tools can help you create a display filter based on the current packet or to investigate a conversation that spans multiple packets.

[^filters]: Do not confuse display filters and capture filters. They serve very different purposes and even use distinct syntax. 

# Instructions
## DHCP
### Setup

Stop any ongoing captures and configure a new capture on your wireless interface with capture filter `arp || udp portrange 67-68`. This filter will prevent Wireshark from capturing excessive amounts of traffic so that we can leave the capture running for the entire lab.

While our current capture should be fairly quiet owing to the previous capture filter, create a display filter to further reduce the number of visible packets (Hint: Type `dhcp` into the display filter input below the main wireshark toolbar).

*At the end of the lab, save your capture and submit it with your assignment.*

### Request a new DHCP Lease

In order to properly complete this lab, you will need to terminate your current DHCP lease so that you can capture the process of DHCP initializing your network interface. This process will differ depending on your OS. 

Follow the instructions linked on [the resources site](/resources/host-config/#renewing-dhcp-leases) to complete this process.

### Report

1.  Based on your capture, create an outline of your complete DHCP exchange. <u>For each message:</u>
    *   Include a screenshot of the packet details summary.
    *   Include a brief written summary, identifying: DHCP message type, DHCP transaction ID, source and destination Ethernet addresses, source and destination IP addresses, and UDP source and destination UDP ports associated.
1.  How can you determine which messages were sent by your own device versus the DHCP server?
1.  Based on the outline you created above, determine the purpose of the 0.0.0.0 address appearing within the DHCP exchange.
1.  What Ethernet-layer (Layer 2) address corresponds to the 255.255.255.255 (Layer 3) IP address? Using online resources as necessary, determine the purpose of these special addresses.
1.  Which of the fields that you have observed can be used to distinguish between different conversations between the same client and server?

If you look closely at the packet details for each message, you'll notice that each DHCP message contains a variety of options. In DHCP, options allow DHCP clients and servers to adapt the protocol to their own needs. Like many other protocols, DHCP encodes options as a *type*, a *length*, and a *value*. Based on this structure, we can continue to extend the protocol without needing to change the basic foundation.

6.  Using your search engine of choice, identify the IETF RFC (# and name) that defines the common option types for the DHCP protocol.
1.  Which DHCP Option (#) tells us the DHCP message type? How many bytes are required to encode the type?
1.  Which DHCP Option (#) tells us the length of a DHCP lease? How many bytes are required to encode the duration of the lease?
1.  According to the RFC, is it possible to specify the lease length in any unit of time other than seconds?
1.  Compare the Lease Time option of the Discover message with the value in the Offer message. Did your device request a particular duration? If so, did the server honor this request?
1.  Using online resources as needed, determine what happens at the end of a DHCP address lease.
1.  Based on the order of the messages in the capture and the details we've asked you to examine, offer an explanation of how the DHCP protocol works to configure your network interface with appropriate settings. How does the protocol make use of Ethernet and IP broadcast functionality, and why is that important? Use your capture to illustrate your point.

## ARP
### Setup

You may continue to use the same capture for this section. If you launch a new capture, please save the first capture and submit both with your assignment.

Before you begin the analysis questions, clear the ARP cache per instructions linked [on the resources site](/resources/host-config/#managing-arp-cache) and generate some Internet traffic, e.g., browse the web.

Remove any existing display filters and create a new filter that will only show ARP related traffic. (Hint: type `arp` in the display filter input).

### Report

13. Outline a complete ARP exchange (request and response). <u>For each message:</u> 
    - Include a screenshot of the packet details summary.  
    - Include a brief written summary, identifying: ARP Opcode, Sender and Target Ethernet addresses, Sender and Target IP addresses.
14. Identify the 48-bit hardware address associated with your network interface and use this information to specify which message(s) originated from your local device?
15. Determine which messages are sent to a broadcast destination.
16. Why do you think the ARP protocol makes use of the broadcast destination?
17. Aside from the broadcast address, what other differences do you notice between the contents of ARP requests and replies?
18. Spend a moment comparing the overall packet structure (as seen in the Packet Details view) between DHCP and ARP. Identify any *layers* that are present in one protocol but not in the other.
19. In order for a network message to be considered an IP packet, the message must contain an Internet Protocol layer that includes an IP header for the rest of the message. Does the ARP protocol use IP packets to communicate?
20. Describe in your own words how ARP works to allow hosts to communicate in a Layer-2 network based on IP address. Use your capture to illustrate your points.