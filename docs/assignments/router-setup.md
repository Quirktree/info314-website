# Build a NAT - enabled Router

## Overview
This assignment builds on the DHCP server functionality you set up in the previous project. Follow the instructions provided here to set up NAT and enable packet forwarding on your Pi. At the end of the project, you will have a working network gateway.

!!! note
    With each checkpoint, we expect you to do more of the work on your own. While the initial Pi setup gave clear instructions, we are slowly moving toward giving you specifications and additional context that you need in order to determine the right steps to accomplish the task.

## Before you Start
Before you begin, make sure that you have completed all steps from Checkpoints #1 and #2 successfully. By now, your pi should be able to connect to the Internet via wireless, and it should provide DHCP services via a pre-defined address range on the ethernet-based LAN.

At times in this project, you will need to refer back to the configuration you defined in the previous **LAN Planning Exercise**. 

## Objectives

In this assignment, you will be configuring the Pi as a simple **Internet gateway** that routes traffic between an isolated, wired LAN and an Internet-connected wireless network. Practically speaking, you'll be tethering your computer to the Pi and using it to access the rest of the Internet.

Whether it is immediately obvious or not, this architecture has practical applications. It is not always a good idea to connect your computer directly to an untrusted network. With a few more components, the Pi provides a simple firewall that can protect against many types of threats. Further, the Pi can be used to tunnel your traffic through a personal VPN, ensuring that that it cannot be intercepted or tampered with on a public network or local ISP.  

### Project outline
Below is a quick outline of what each part of the project will look like.
1.	Complete and test the DHCP configuration (last project)
2.	Configure basic firewall rules and NAT
3.	Enable packet forwarding settings within Raspbian
4.	Update DHCP to provide gateway and DNS settings



## What is NAT
We will leverage network address translation (NAT) so that our hosts (using RFC 1918 addresses) can access public networks. In this scenario, your Pi will be translating source addresses from the LAN to masquerade with the IP address assigned to the WAN port. 

This setup is a special case of source NAT (SNAT) in that many private IP addresses will be mapped to one public address on the external interface. To accommodate, the Pi will need to monitor the state of each connection and other network traffic so that it can route to the correct host on the internal network when incoming traffic is received. 

## Configuring NAT
NAT functionality is supported natively in Linux through the **netfilter firewall**. We'll use a common Linux tool known as **iptables** to manage this mechanism. Iptables defines rules in a hierarchical structure that are used to filter and manipulate packets. 

!!! Attention
    Throughout the instructions, we'll refer to the wired interface as **&lt;LAN&gt;** (because it serves our local network) and the wireless interface as the **&lt;WAN&gt;** (because it connects us to the Internet). Your configuration files will reflect the Raspbian-assigned interface names, such as: eth0 and wlan0.

We have prepared an [**iptables** tutorial](../iptables-nat) that focuses specifically on the features that are required to implement NAT. Read this tutorial first, and then use it as a reference to fill in the missing values in the starter template provided for this project. You will find a copy of the starter template inside your GitHub project repository under **resources/iptables.conf.template**. 

!!! Attention "Copy/Paste Warning"
    This template has missing values that you must define before trying to load on your Pi.

### Rule Specification
The template provided already provides most of what is needed to establish a basic NAT configuration. Fill in the missing values and ensure that your final ruleset complies with the following specifications.

* Append a new rule to the `POSTROUTING` chain that masquerades outbound packets on the WAN interface.
* Set the default policy for the `FORWARD` chain of the filter table to drop all packets.
* Create a rule that accepts outbound packets being received on the LAN interface and sent to the WAN.
* Create a rule to accept inbound packets that are related to existing outbound connects or part of other established connections.

While best practice would have us further configure the firewalls for the internal and external interfaces on the Raspberry Pi, we will save this step for a future assignment.

### Testing and Applying Rules
After you finish filling out the missing values in the reference template according to the above specifications, you should follow the instructions provided in the [provided tutorial](../iptables-nat/#loading-rules-from-the-file-system-with-iptables-persistent) to set up your rules to be loaded persistently.

!!! note
    You will not be able to completely test your rules until you complete the upcoming steps. What you should ensure at this point is that the rules are applied and that you are not locked out of the Pi.

## Enable Packet Forwarding
Once you are confident that you have set up NAT and filtering properly within iptables, you need to instruct the Linux kernel to begin forwarding packets. 

You can enable the setting temporarily from the command line by calling `sudo sysctl -w net.ipv4.ip_forward=1`, but it will not persist across reboots. 

Make this change persistent inside `/etc/sysctl.d/99-sysctl.conf` by setting `net.ipv4.ip_forward=1` (the rule is already included in the file but commented out).

---
## Update DHCP Configuration

Update the DHCP configuration written in the previous assignment to provide clients with settings for the **default router** and a DNS resolver (called **name servers** in **isc-dhcp-server**). Use the examples provided in the default `dhcpd.conf` as the basis for your changes.

## Test your Configuration
You should be able to access external networks by way of the Raspberry Pi. Disable your main wireless or wired network so that the Pi is your only route to the Internet. Try connecting to a well-known website. From the command line (your laptop), run a `traceroute` (`tracert` on Windows) to confirm that the Pi is providing a route.

If you run into problems here, there are a few points to check. First, try to ping a known address such as 1.1.1.1 from your computer. This will tell you whether or not you have connectivity outside your network. If you're at UW, try pinging 128.95.112.1, to determine whether you can access hosts on campus. You may also try pinging a known domain name like amazon.com (and washington.edu if you're on campus).

* If you can't ping anything, you may have an issue with the network configuration on your computer. Try pinging the static address you created for the Pi. If this fails with no route to host, go back and make sure you set up the Pi and your computer correctly.
* If you can ping by IP address, but not name, you have an issue with DNS. Both Apple and Microsoft will allow you to add a static DNS server to a network interface. Check that you've done this as described above.
* If you're able to ping your Pi by it's IP and you're certain that you've set up your local network correctly, go back and confirm that the Pi has forwarding enabled for IP and that your iptables rules are loading. 
