# Basic IP Address Planning

Don't try to build a network without a plan. A good network design is built on parameters and constraints that are appropriate for the intended use-cases. 

This guide offers a generic procedure that can serve as a minimal starting point for planning a simple network or subnet. 

## Address Types

**Leased Addresses** When a new computer joins 
**Static Addresses** 
**DHCP Reservations**



## Procedure

### Step 1 - Examine address needs

Determine the number of addresses required, accounting for static, reserved, and DHCP address pools.

Identify additional address constraints, such as the number of distinct subnets needed or potential subnet conflicts

### Step 2 - Identify the base network

Identify your base address block or select a block from the RFC-1918 address range.

### Step 3 - Select subnet parameters 

Determine the CIDR length for your subnet and the subnet mask for your LAN.

Based on your selected RFC-1918 address block, CIDR length, and additional constraints, select a subnet that will be used to allocate addresses for your LAN.

Based on the previous values, determine the broadcast address associated with your network.

### Step 4 - Document important addresses and ranges

Assign any static addresses that are needed to provide core network functions and other services.

Define the range of addresses available for use in the DHCP address pool.
