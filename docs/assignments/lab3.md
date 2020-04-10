# Lab 3 - Analyze Zeroconf in Wireshark

[Lab 3 Assignment page on Canvas](https://canvas.uw.edu/courses/1373089/assignments/5369619)

## Overview

Most of the time, our discussion about IP addresses are focused on globally routable addresses or on RFC1918 private addresses. In this lab, we are going to explore another type of address that can be used without any explicit configuration (either by a user or network administrator). We’re also going to look at how these addresses fit in to a group of protocols called the zero-configuration networking (zeroconf) protocols and explore how zeroconf appears in this class and in our daily use of network technology.

This lab will require a combination of Wireshark analysis and Internet research. We’ve provided links to several references as a starting point, but you are free to look up other resources as needed. 

*Please cite all sources in your deliverable.*

## Resources

[https://en.wikipedia.org/wiki/Link-local_address (Links to an external site.)](https://en.wikipedia.org/wiki/Link-local_address)

[https://tools.ietf.org/html/rfc3927 (Links to an external site.)](https://tools.ietf.org/html/rfc3927)

[https://en.wikipedia.org/wiki/.local (Links to an external site.)](https://en.wikipedia.org/wiki/.local)

[https://en.wikipedia.org/wiki/Zero-configuration_networking (Links to an external site.)](https://en.wikipedia.org/wiki/Zero-configuration_networking)


## Part I - Identification

**Report:**

1. Before you start, take a moment to record some details about your computer and your Ethernet network adapter. These details may be helpful to you in your analysis:
    - Hostname (Computer, Pi)
    - MAC Address (Computer, Pi)


## Part II - Powering on your Pi

If you already started your Pi, shut it down and disconnect power before you begin.

Launch a new Wireshark capture over the Ethernet connection to your Pi. Linux users should listen on the “All” interface and may want to disable their wireless connection before capturing to reduce noise and focus on the important details.

With your capture running, power on the Pi. You’ll be keeping an eye on the capture in order to see what happens when your computer and your Pi first connect to the network.

**Report:**

2. Identify the link-local addresses and mDNS names associated with your computer and your Pi.

3. Using your capture as a reference, identify the messages that your Pi sends while autoconfiguring its own IP addresses.

4. Describe the autoconfiguration process used by hosts to choose a link-local address and confirm that it is unique on the local network segment.


## Part III - Contacting your Pi

With your capture still running, let’s try to contact your Pi. Try pinging your Pi based on the hostname “raspberrypi.local” (substitute your own hostname if you’ve already changed it). Likewise, try connecting to your pi via SSH.

**Report:** 

5. Using your capture, identify an example of an IPv4 and IPv6 mDNS request.

6. Identify the multicast addresses used by mDNS in IPv4 and IPv6.

7. Using your capture, identify an example of an IPv4 and IPv6 mDNS response.

8. Are mDNS responses sent to a multicast address or to a specific unicast address.

9. Identify the destination addresses used to reach your Pi in the final ICMP echo request (ping) and SSH connection.

## Part IV - Manually Querying mDNS devices

Lastly, let’s look at some tools that will allow us to manually query mDNS devices on our network. Unlike DNS, we should see that individual devices answer their own queries. Windows users can use the powershell command `Resolve-DNSName` while macOS and Linux users should be able to use `dns-sd`.

**Report:**

10. Use `man` to look up more information about the relevant command and to determine the correct syntax for making an mDNS request to your Pi. Provide a screenshot of your mDNS request and the output received.

## Part V - Protocol Research

Perform your own research into these protocols and answer the following questions. We recommend starting with the resources listed at the end of this guide.

11. Describe the major difference between how link local addresses are assigned and used in IPv4 versus in IPv6.

12. In your own words, summarize how RFC 3927 says we can determine whether hosts are located on the same network link.

13. Based on what you’ve learned, describe how we are able to connect with a Pi based on a pre-defined hostname without the need for any infrastructure like a DHCP or DNS server.

14. Based on what you’ve learned about link-local addresses, describe how you might use Wireshark in conjunction with other network utilities to troubleshoot the connection to a headless device (i.e., a device like the Pi that you can only access over the network).

## Extra Credit (Optional)

15. Perform an mDNS scavenger hunt with Wireshark on a couple of different networks. Provide a short (about ½ page) analysis of your search along with capture samples. What types of devices were you able to discover? How does mDNS make your life more convenient?