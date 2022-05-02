# Build a NAT-enabled Router (2022-04-20)

## Overview
This assignment builds on the DHCP server functionality you set up in the previous project. Follow the instructions provided here to set up NAT and enable packet forwarding on your Pi. At the end of the project, you will have a working network gateway.

!!! note
    With each checkpoint, we expect you to do more of the work on your own. While the initial Pi setup gave clear instructions, we are slowly moving toward giving you specifications and additional context that you need in order to determine the right steps to accomplish the task.

---
## Before you Start
Before you begin, make sure that you have completed all steps from Checkpoints #1 and #2 successfully. By now, your pi should be able to connect to the Internet via wireless, and it should provide DHCP services via a pre-defined address range on the ethernet-based LAN.

At times in this project, you will need to refer back to the configuration you defined in the previous **LAN Planning Exercise**. 

---
## Objectives

In this assignment, you will be configuring the Pi as a simple _Internet gateway_ that routes traffic between an isolated, wired LAN and an Internet-connected wireless network. Practically speaking, you'll be tethering your computer to the Pi and using it to access the rest of the Internet.

Whether it is immediately obvious or not, this architecture has practical applications. It is not always a good idea to connect your computer directly to an untrusted network. With a few more components, the Pi provides a simple firewall that can protect against many types of threats. Further, the Pi can be used to tunnel your traffic through a personal VPN, ensuring that that it cannot be intercepted or tampered with on a public network or local ISP.  

### Project outline
The main steps of this project are listed below:

1.	Configure basic firewall rules and NAT
1.	Enable packet forwarding settings within Raspberry Pi OS
1.	Update DHCP to provide gateway and DNS settings
1.  Test and troubleshoot your configuration

---
## Overview of NAT
The hosts in our network use _RFC-1918_ addresses, which are restricted to private networks and cannot be used as a source or destination for addresses on packets being sent over the Internet. We'll address this problem with _network address translation (NAT)_, which will allow your Pi to communicate on the Internet by masquerading with the public address assigned to the WAN port of the router.

This setup is a special case of _source NAT_ in that many private IP addresses will be mapped to one public address on the external interface. To accommodate, the Pi will need to monitor the state of each connection and other network traffic so that it can route to the correct host on the internal network when incoming traffic is received. 

## Configuring NAT
_NAT_ functionality is supported natively in Linux through the _netfilter firewall_ and can be managed through a framework known as _nftables_ by defining rules that filter and manipulate packets. 

!!! Attention
    Throughout the instructions, we'll refer to the wired interface as **&lt;LAN&gt;** (because it serves our local network) and the wireless interface as the **&lt;WAN&gt;** (because it connects us to the Internet). Your configuration files will reflect the OS-assigned interface names, such as: _eth0_ and _wlan0_.

We have prepared an [nftables tutorial](/resources/nftables) that focuses specifically on the features that are required to implement _NAT_. Read this tutorial carefully before you attempt to configure _nftables_ on your Pi.

### Rule Specification
The previous guide contains a template and partial ruleset to help you enable _NAT_ and restrict traffic forwarded between networks. Save a copy of this template in the home directory of your Pi and fill in the missing values so that it complies with the following requirements:

* Create a table named `filter` with a base chain named `forward` that is attached to the `filter`/`forward` hook.
* Use a default policy to drop packets on the `forward` chain unless they're explicitly allowed by another rule.
* Include a rule to allow all _outbound_ packets on the `forward` chain.
* Include a rule to allow _inbound_ packets on the `forward` chain if they are related to previous packets or established connections.
* Create a table named `nat` with a base chain named `postrouting` that is attached to the `nat`/`postrouting` hook.
* Include a rule to masquerade the addresses for _outbound_ packets on the **&lt;WAN&gt;** interface.

### Testing and Applying Rules
After you finish filling in the missing values in the reference template according to the above specifications, follow the instructions in the [provided tutorial](/resources/nftables/#editing-nftable-rules-safely) to test your rules and load them at boot.

You will not be able to completely test your rules until you complete upcoming steps. What you should ensure at this point is that the rules are applied and that you are not locked out of the Pi.

---
## Enable Packet Forwarding
With your _nftables_ configuration in place, you are ready to enable _forwarding_ within the OS. By default, the Linux networking stack accepts packets with its own IP addresses and discards everything else. This behavior can be changed through the _sysctl_ service, enabling the network stack to _forward_ packets that are destined for nodes on another network. 

To turn on IPv4 forwarding when the system boots, add a new file named `/etc/sysctl.d/10-forward_ip4.conf` containing `net.ipv4.ip_forward=1`. After saving this file, call `sudo sysctl -p --system`.

---
## Update DHCP Configuration

Update the DHCP configuration written in the previous assignment to provide clients with settings for a _default router_ and a public DNS _resolver_ (called _domain name servers_ in **isc-dhcp-server**). 

* Use the examples provided in the default `dhcpd.conf` as the basis for your changes.
* Restart **isc-dhcp-server** in order to load the new configuration.
* Renew your DHCP lease by temporarily disconnecting Ethernet or following [instructions](/resources/manage-dhcp/) provided in the resources section of this site.

---
## Test your Configuration
Once you have completed these changes, you should be able to access external networks by way of the Raspberry Pi.

1. Disable wireless networking and other network interfaces on your laptop so that the Pi is your only route to the Internet. 
1. Try connecting to a well-known website from your browser. 
1. From your laptop's command line, use the `traceroute` command (`tracert` on Windows) to confirm that your Pi is the first hop of this route.

---
## Troubleshooting
If you run into problems here, there are a few points to check. First, try to `ping` a known address such as 1.1.1.1 from your computer. This will tell you whether or not you have connectivity outside your network. If you're at UW, try pinging 128.95.112.1, to determine whether you can access hosts on campus. You may also try pinging a known domain name like amazon.com (and washington.edu if you're on campus).

* If you can't ping anything, you may have an issue with the network configuration on your computer. Try pinging the static address you created for the Pi. If this fails with no route to host, go back and make sure you set up the Pi and your computer correctly.
* If you can ping by IP address, but not name, you have an issue with DNS. Verify that DHCP is providing a valid name server address. 
* If you're able to ping your Pi by it's IP and you're certain that you've set up your local network correctly, go back and confirm that the Pi has forwarding enabled for IP and that your iptables rules are loading. 
