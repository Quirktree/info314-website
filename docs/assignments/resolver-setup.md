# Configure a Recursive DNS Resolver (last edited 2022-04-27)
## Overview
In this assignment, we'll continue to extend the functionality of the Linux-based router by installing and configuring the open source BIND server to resolve DNS requests on behalf of hosts on your LAN. This new functionality will take the place of the public DNS resolvers we used in the previous assignments (e.g., `1.1.1.1` or `8.8.8.8`).

!!! important
    With each checkpoint, we expect you to do more of the work on your own. While the initial Pi setup gave clear instructions, we are slowly moving toward giving you specifications and additional context that you need in order to determine the right steps to accomplish the task.

    In this assignment, you'll be pointed toward external sources that you can reference in order to implement the project specifications. This can be a challenging adjustment, but it is also a valuable skill to learn and practice as you prepare for technical internships and jobs.

## Before you start
Before you begin, make sure that you have completed all steps through Checkpoint #3 successfully. At this point, your Pi should be able to route traffic between the Ethernet LAN and a public-facing Wireless network. Likewise, DHCP should provide LAN clients with a full network configuration, including a default gateway and a public DNS resolver. As with the previous checkpoint, you will rely on parameters you determined in your **LAN Planning Exercise** to complete this project.

## Instructions

### Review your Address Plan

Running the DNS resolver privately will require an IP address from your network range available on the Pi. While many home router configurations will use a single address for DHCP, routing, and DNS resolution, the specifications for this project require you to use a separate IP address that you selected when creating your initial address plan. Review that plan now.

### Configure the Resolver's IP Address

Before you configure a new service with an address, you need to set up the address at the device level. Add your resolver's address to the static configuration for _eth0_. In a _networkd_ configuration, you can add additional addresses by putting each address (in abbreviated CIDR format) on a separate `Address=` line within the _.network_ file. In order to do this, you may want to review the original setup given in [Checkpoint #2](../dhcp-setup/#configure-static-addresses-for-your-raspberry-pi).

### Install BIND on the router

The _Berkeley Internet Name Daemon (BIND)_ is a robust and versatile DNS implementation distributed by the *Internet Services Consortium (ISC)* -- the same folks responsible for the DHCP server implementation we have deployed. The first version of _BIND_ was released in 1986. At present, _BIND 9_ is the most widely deployed DNS server on the Internet.

_BIND_ and related utilities are packaged in the Debian and Raspberry Pi OS software repositories, meaning that they can be installed and managed easily with the `apt` utilities that we are already familiar with. For our purposes, you should install the following packages: `bind9`, `bind9utils`, `bind9-doc`.

### Configure BIND as a Recursive Name Server (i.e., a Resolver)

_Name servers_ or _DNS servers_ come in multiple flavors that are differentiated based on the queries they're configured to answer.[^server-types] Some servers are responsible for a single tier or branch of the DNS hierarchy, e.g., names within the _tcpip.dev_ domain, while other servers offer the ability to answer queries about any domain. _BIND_ can be configured to operate in all of these roles, but in this checkpoint, we're focused on the latter objective. 

Until now, devices in your network relied on public servers to resolve _recursive DNS queries_, but you can configure _BIND_ to act as the resolver for clients connecting from your LAN. The requirements for this task are described below, but you will likely need other resources in order to implement these specifications.

There are a variety of tutorials available online, but [Digital Ocean's community tutorial for Ubuntu Linux](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-caching-or-forwarding-dns-server-on-ubuntu-16-04) is more consistent than most with respect to project requirements. This guide does a good job explaining the organization of the configuration files you'll encounter and the syntax used when adding directives like `acl` or `allow-query`. Pay special attention to the _caching DNS server_ section of this guide. The _forwarding DNS server_ instructions are not relevant to this project.

[^server-types]: See [A Comparison of DNS Server Types](https://www.digitalocean.com/community/tutorials/a-comparison-of-dns-server-types-how-to-choose-the-right-dns-configuration)

#### Requirements

1. Use the `acl` directive to create an access control list (ACL) that restricts access to BIND to devices on your _LAN_ (your subnet range in the CIDR format) in addition to the Pi itself (using the `localhost` keyword).
    * If you're curious to learn more about this security precaution, read up on _Cache Poisoning_ and _DNS Amplification_ or hit us up in office hours.
1. Enable `recursion` and use the `allow-query` directive to grant access through your access control list 
1. Limit which IPv4 addresses on the Pi will be listening for DNS queries by configuring a `listen-on` block with the following local addresses:[^listen-on] 
    * _`127.0.0.1`_ is the _loopback_ address, allowing the Pi to use BIND to answer its own queries.
    * The static IP configured earlier for the DNS resolver.
1. Disable IPv6 by changing `listen-on-v6 { any; };` to `listen-on-v6 { none; };`.
1. Disable DNSSEC functionality by replacing the configuration line that reads `dnssec-validation` from `auto` to `no`.
    * Disabling DNSSEC is necessary due to the limitations of using a Pi versus an always on network server.
    * Specifically, the inability of the Pi to maintain accurate time when it is powered off interferes with the verification process for DNSSEC cryptographic signatures

[^listen-on]: This directive is missing from the original file, but you can add it immediately before or after `listen-on-v6`.

### Test and verify that the new server is operating properly
In addition to the troubleshooting tools that we're already familiar with. BIND comes with a utility called `named-checkconf` that will review its core configuration files for syntax errors. It's important to run this tool each time you modify the BIND configuration in this assignment or future checkpoints. If it doesn't find any syntax errors in your configuration files, `named-checkconf` will return without displaying any messages. 

Once your files are free of errors, use `systemctl` to restart the `bind9` service and check that it's working with `dig`, e.g., `dig @127.0.0.1 washington.edu`. As we've discussed in other places, the `@IP` syntax of `dig` is used to override local resolver settings (which are still determined by DHCP on the wireless network).

### Update your DHCP Server Configuration

To wrap up this project, you need to update the DHCP server configuration so that LAN users will receive the address for the local resolver instead of the public resolver we previously used:

1. Update DHCP server configuration so that devices configured by DHCP will use the Pi's DNS resolver rather than the public server you configured in Checkpoint #3.
1. Use `systemctl` to restart the DHCP server daemon.
1. Manually renew your DHCP configuration on your PC through the OS or by temporarily disconnecting the network cable.
1. Verify that your computer receives the updated parameters from DHCP.