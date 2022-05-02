# Basic IP Address Planning (2022-04-04)

Don't try to build a network without a plan. A good network design is built on parameters and constraints that are appropriate for the intended use-cases. This guide offers a simple procedure that can serve as a starting point for basic planning with a single subnet network.

## Private versus Public

We can't pull addresses from thin air. The laws of the TCP/IP universe restrict what addresses we can use on networks we can control. Make sure that you understand where addresses come from and can distinguish between private and public address space.

**Private Address Ranges** Within the boundaries of our own network, we are relatively free to set our rules as long as we use addresses that are reserved for private use. Your reference point for these addresses is [RFC-1918](https://tools.ietf.org/html/rfc1918).

**Public Address Ranges** For packets we send on the public Internet, we must use public addresses that are in our control. These addresses will either be leased (short-term or long-term reservation) to us by an ISP or allocated on a more permanent basis by a Regional Internet Registry (RIR)[^rir]. A contiguous group of addresses allocated in this way is referred to as a **block**.

[^rir]: More information about RIRs can be found at https://www.nro.net/about/rirs/.

**Special Address Ranges** Beyond these two ranges, we frequently encounter other address ranges that are reserved for special use-cases, e.g., the loopback (127.0.0.0/8) and link-local (169.254.0.0/16) address ranges.
[RFC-6890](https://datatracker.ietf.org/doc/html/rfc6890)

## Configuration Strategies

As you get started, you will need to make decisions about how you will configure devices on your network with appropriate IP addresses and network parameters. Before you proceed, make sure that you understand the following classifications.

**Leased Addresses** As much as possible, you'll want to avoid the need for manual configuration when a new device joins the network. DHCP allows you to allocate a range of addresses that the server can use to issue address leases to hosts that request them. Leased addresses are dynamic in the sense that they aren't determined until a host joins the network and initiates the DHCP process.

**Static Addresses** Dynamic address assignment won't work for every scenario, e.g., when an address should be fixed and a dependency on DHCP isn't possible. Some devices, including your network gateway and the DHCP server itself should be manually configured. We describe their addresses as static because they are defined through the network node's own configuration and won't change based on external factors.

## Procedure

### Step 1 - Examine address needs
Determine the size of your network, i.e., the number of devices that will need addresses at any given point in time, and document the types of devices you expect to service as well as their address requirements. Don't forget to account for network infrastructure and additional growth.

Not all devices on a network are the same, and so you will often have clients with different address needs. End users on their computers and mobile devices will almost always receive dynamic (leased) addresses, but you might need to reserve some address space for static addresses to acommodate web servers, mail servers, or even (sometimes) printers.  

Your network infrastructure consists of routers, essential services like DHCP and DNS, as well as managed switches and access points. In very simple networks, these services may live on the same device and share a single IP address. More sophisticated networks will split these services onto dedicated devices, or at the very least, assign them dedicated IP addresses.

Always assume that your network will grow over time. Allocate additional address space in anticipation of this growth. If you have believe that you will need 30 addresses today, it's probably safer to plan for at least 60 addresses.

### Step 2 - Identify the base address range

Identify the base address range from which you will be subnetting. If you do not have a permanent allocation, select a block from the RFC-1918 address range.

### Step 3 - Select subnet parameters 

Based on the inventory you conducted in step #1, determine the number of address bits that are required in order to give an address to each host on your LAN. You can use this number to determine the CIDR length and network mask for your Subnet.

Given a base network and a CIDR length, choose an available subnet id for your network.[^prefix]

To complete the set of basic configuration parameters, compute the broadcast address associated with your chosen subnet ID and CIDR.

[^prefix]: The subnet ID or network prefix is the portion of the address that stays the same for all hosts on the same layer-2 network segment. 

### Step 4 - Document important addresses and ranges

Finally, identify how you plan to allocate the addresses in your network among devices based on the requirements you identified in Step #1. 

Static addresses will usually be assigned to core network devices, such as your router, DHCP, and DNS servers. If you identified any other devices that need consistent addresses, assign those addresses now.

When you operate multiple networks, it is helpful to be consistent with your static assignments. For example, most network administrators will assign the router to either the first or last address in the subnet. Don't deviate from convention unless you have a good reason.

Once you've documented your static addresses, identify a range of addresses within your subnet that can be used for the DHCP address pool. This should be a contiguous block of addresses that does not overlap with your a) network ID, b) subnet's broadcast address, or c) static addresses that you already defined.
