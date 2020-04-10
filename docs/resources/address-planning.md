# Basic IP Address Planning (2020-04-09)

Don't try to build a network without a plan. A good network design is built on parameters and constraints that are appropriate for the intended use-cases. This guide offers a simple procedure that can serve as a starting point for basic planning needs.

## Private versus Public

We can't pull addresses from thin air. The laws of the TCP/IP universe restrict what addresses we can use on networks we can control. Make sure that you understand where addresses come from and can distinguish between private and public address space.

**Private Addresses** Within the boundaries of our own network, we are relatively free to set our rules as long as we use addresses that are reserved for private use. Your reference point for these addresses is [RFC-1918](https://tools.ietf.org/html/rfc1918).

**Public Addresses** For packets we send on the public Internet, we must use public addresses that are in our control. These addresses will either be leased (short-term or long-term reservation) to us by an ISP or allocated on a more permanent basis by a Regional Internet Registry (RIR)[^rir]. A contiguous group of addresses allocated in this way is referred to as a **block**.

[^rir]: More information about RIRs can be found at https://www.nro.net/about/rirs/.

## Configuration Strategies

As you get started, you will need to make decisions about how you will configure devices on your network with appropriate IP addresses and network parameters. Before you proceed, make sure that you understand the following classifications.

**Leased Addresses** As much as possible, you'll want to avoid the need for manual configuration when a new device joins the network. DHCP allows you to allocate a range of addresses that the server can use to issue address leases to hosts that request them. Leased addresses are dynamic in the sense that they aren't determined until a host joins the network and initiates the DHCP process.

**Static Addresses** Dynamic address assignment won't work for every scenario, e.g., when an address should be fixed and a dependency on DHCP isn't possible. Some devices, including the DHCP server itself should be directly configured. We describe their addresses as static because they are coded into a network node's configuration and won't change base on external factors.

## Procedure

### Step 1 - Examine address needs

Determine the number of addresses required, accounting for static, and DHCP Leased address pools.

Identify additional address constraints, such as the number of distinct subnets needed or potential subnet conflicts

### Step 2 - Identify the base network

Identify the base network from which you will be subnetting. If you do not have a permanent select a block from the RFC-1918 address range.

### Step 3 - Select subnet parameters 

Based on the inventory you conducted in step #1, determine the number of address bits that are required in order to give an address to each host on your LAN. You can use this number to determine the CIDR length and network mask for your Subnet.

Given a base network and a CIDR length, choose an available subnet id for your network.[^prefix]

To complete the set of basic configuration parameters, compute the broadcast address associated with your chosen subnet ID and CIDR.

[^prefix]: The subnet ID or network prefix is the portion of the address that stays the same for all hosts on the same layer-2 network segment. 

### Step 4 - Document important addresses and ranges

Finally, let's connect the dots between what we've done in the first three steps by splitting your subnet up according to static, and dynamic addresses.

Static addresses will usually be assigned to core network devices, e.g., a router or DHCP server, that should operate independently of DHCP. For other devices that need consistent addresses -- perhaps DNS servers or printers -- you can also assign a static address.

If you operate multiple networks, it is helpful to be consistent with your static assignments. For example, most network administrators will assign the router to either the first or last address in the subnet. Don't deviate from convention unless you have a good reason.

Once you've documented your static addresses, identify a range of addresses within your subnet that can be used for the DHCP address pool. This should be a contiguous block of addresses and it cannot overlap with your a) network ID, b) subnet broadcast address, or c) static addresses that you've defined.
