# Basic IP Address Planning (2020-01-18)

Don't try to build a network without a plan. A good network design is built on parameters and constraints that are appropriate for the intended use-cases. This guide offers a simple procedure that can serve as a starting point for basic planning needs.

## Address Types

**Leased Addresses** As much as possible, you'll want to avoid the need for manual configuration when a new device joins the network. DHCP allows you to allocate a range of addresses that the server can use to issue address leases to hosts that request them. Leased addresses are dynamic in the sense that they aren't determined until a host joins the network and initiates the DHCP process.

**Static Addresses** Dynamic address assignment won't work for every scenario, e.g., when an address should be fixed and a dependency on DHCP isn't possible. Some devices, including the DHCP server itself should be directly configured. We describe their addresses as static because they are coded into a network node's configuration and won't change base on external factors.

**DHCP Reservations** Static address assignment provides a maximum degree of control when we want a device to stay at a fixed address, e.g., servers and printers, but introduces additional management cost since devices have to be configured on an individual basis. With a reservation, we can register a device based on its MAC address and ensure that DHCP serves it a pre-defined configuration when it joins the network. 

## Procedure

### Step 1 - Examine address needs

Determine the number of addresses required, accounting for static, reserved, and DHCP address pools.

Identify additional address constraints, such as the number of distinct subnets needed or potential subnet conflicts

### Step 2 - Identify the base network

Your network/subnet addresses or select a block from the RFC-1918 address range.

### Step 3 - Select subnet parameters 

Determine the CIDR length for your subnet and the subnet mask for your LAN.

Based on your selected RFC-1918 address block, CIDR length, and additional constraints, select a subnet that will be used to allocate addresses for your LAN.

Based on the previous values, determine the broadcast address associated with your network.

### Step 4 - Document important addresses and ranges

Assign any static addresses that are needed to provide core network functions and other services.

Define the range of addresses available for use in the DHCP address pool.
