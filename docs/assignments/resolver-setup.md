# DNS Resolvers (last edited 2020-02-09)
## Overview
In this assignment, we'll continue to extend the functionality of the Linux-based router by installing and configuring the open source BIND server to resolve DNS requests on behalf of hosts on your LAN. This new functionality will take the place of the public DNS resolvers we used in the previous assignments (e.g., `1.1.1.1` or `8.8.8.8`).

!!! important
    With each checkpoint, we expect you to do more of the work on your own. While the initial Pi setup gave clear instructions, we are slowly moving toward giving you specifications and additional context that you need in order to determine the right steps to accomplish the task.

    In this assignment, you will use an [existing tutorial](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-caching-or-forwarding-dns-server-on-ubuntu-16-04) as a reference to **help** you complete the project specifications. Please note that you will not follow the tutorial verbatim, but instead make adjustments based on the project requirements.

    For some students, this can be a challenging adjustment, but it is also a valuable skill to learn and practice as you prepare for technical internships and jobs.

## Before you start
Before you begin, make sure that you have completed all steps through Checkpoint #3 successfully. At this point, your Pi should operate as a NAT-router between the Ethernet LAN and a Wifi-based WAN. Likewise, DHCP is configured to provide LAN clients with a full network configuration, including a default gateway and a public DNS resolver.

At times in this project, you will need to refer back to the configuration you defined in the previous **LAN Planning Exercise**. 

## Instructions

### Review/Update your IP Address Plan
Since we will be running our DNS resolver inside our LAN, we will need to provide it with an IP address on *eth0*. Choose an unused static address from the ranges you defined in your initial LAN Planning. 

Before you can assign this address to a service, you will need to add another `Address=` line to your *eth0* configuration in *systemd-networkd*. Further instructions were provided in [Checkpoint #2](../dhcp-setup/#configure-static-addresses-for-your-raspberry-pi).

While it may not be clear at this point, there are sometimes benefits to using separate addresses for diffferent services (in real-world scenarios) even though they're all running on the same device and network interface. The first benefit that comes to my mind is that planning for multiple addresses sets you up for easy changes to your network architecture in the future. It might make sense to combine all of these services onto one device today, but that might not always be the case depending on your requirements.

### Install BIND on the router
**BIND** (also refered to as *named*) is a robust and versatile DNS implementation developed by the *Internet Services Consortium (ISC)* -- the same folks that developed the DHCP server implementation we have deployed. The first version of BIND was released in 1986. At present, BIND 9 is the most widely deployed DNS server on the Internet.

BIND and related utilities are packaged in the Debian/Raspbian software repositories, meaning that they can be installed and managed easily with the `apt` utilities that we are already familiar with. For our purposes, we want to install three different packages named `bind9`, `bind9utils`, `bind9-doc`. Please refer to previous guides if you need to review the options for the `apt` command.

### Configure BIND as a caching DNS server
The term _DNS server_ can refer to several distinct roles that are defined in the DNS specifications[^server-types]. BIND can be configured to operate in all of these roles, but in this checkpoint, we want to configure the service to handle recursive DNS resolution for our LAN clients. Until now, our clients relied on public DNS services. 

[^server-types]: See [A Comparison of DNS Server Types](https://www.digitalocean.com/community/tutorials/a-comparison-of-dns-server-types-how-to-choose-the-right-dns-configuration)

To accomplish this objective, refer to the section of Digital Ocean's tutorial[^tutorial], which demonstrates how to set up a caching DNS server (be sure to stop after this section) and the notes below.

[^tutorial]: See [How to Configure Bind as a Caching or Forwarding DNS Server](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-caching-or-forwarding-dns-server-on-ubuntu-16-04)

You will need to make several adjustments to the configuration you created based on DO's guidance:

1. Confirm that you have adapted the instructions to use your own IP address ranges.
1. Disable IPv6 by changing `listen-on-v6 { any; };` to `listen-on-v6 { none; };`.
1. Limit which IPv4 addresses will be listening for DNS queries by setting `listen-on { 127.0.0.1; <DNS IP>; }`.
    * **`<DNS IP>`** will be replaced by the static IP on which you want to respond to DNS requests
    * If the `listen-on` block is missing from the original file, you can add it immediately before or after `listen-on-v6`.
    * 127.0.0.1 is the loopback address. We're including this for times that your Pi will query its own resolver.
1. Restrict the `acl` to include `localhost` and your own subnet range (CIDR format)
    * If you added `localnets` while following the tutorial, remove it now. This setting would allow the resolver to be accessed on the external (wireless network) as well as your LAN.
    * If you're curious to learn more about this security precaution, read up on **Cache Poisoning Attacks** or hit us up in office hours.
1. Disable DNSSEC functionality by replacing the configuration line that reads `dnssec-validation auto;` with `dnssec-enable no;`.
    * Disabling DNSSEC is necessary due to the limitations of using a Pi versus an always on network server.
    * Specifically, the inability of the Pi to maintain accurate time when it is powered off interferes with the verification process for DNSSEC cryptographic signatures

### Test and verify that the new server is operating properly
In addition to the troubleshooting tools that we're already familiar with. BIND comes with a utility called `named-checkconf` that will review its core configuration files for syntax errors. You should run it any time you modify the BIND configuration in this assignment or future checkpoints.

!!! faq "Why didn't `named-checkconf` do anything"
    If it doesn't find any syntax errors in your configuration files, `named-checkconf` will return without displaying any messages. To see what sort of errors are reported by the utility, create an intentional syntax error and run the tool again.

Once your files are free of errors, use `systemctl` to restart the `bind9` service and check that it's working with `dig`, e.g., `dig @127.0.0.1 washington.edu`. As we've discussed in other places, the `@IP` syntax of `dig` is used to override local resolver settings (which are still determined by DHCP on the wifi network).

### Update your DHCP Server Configuration

To wrap up this project, you will need to update the DHCP server configuration so that LAN users will receive the address for the local resolver instead of the public resolver we previously used:

1. Update DHCP server configuration so that your Pi will be used as the domain name server for the LAN
1. Use systemctl to restart the DHCP server daemon
1. Manually renew your DHCP configuration on your PC through the OS or by temporarily disconnecting the network cable 
1. Verify that your computer receives the updated parameters from DHCP




