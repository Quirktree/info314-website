# Configure a Routing Link for BGP Peers
## Determine Requirements
In order to configure your router to peer with other BGP routers, you will need to identify several parameters associated with the routing configuration. These parameters can differ somewhat depending on the type of link you use, i.e., physical or tunnel-based, and other constraints. This guide presents two different scenarios: a) VLANs over a physical Ethernet connection and b) WireGuard to establish a tunnel-link between two routers.

In all cases, we need an Autonomous System Number (ASN) for each Autonomous System and we need a unique IP address that will identify each router (regardless of which interface is used). This latter parameter is called the router ID.

For each routing link, we also need to provide a way for the routing peers to deliver packets from host-to-host -- implying the use of IP addresses. Free Range Routing (FRR) supports an "unnumbered" address configuration which saves us the trouble of planning out addresses on the links; however, this configuration depends on link-local IPv6 capabilities that won't work as expected over a WireGuard tunnel. In these cases, BGP can make use of peer-to-peer addresses configured on the WireGuard network interfaces.

!!! "example" Configuration Parameters for VLAN-based peers over Ethernet
    - Interface for routing link: `vlan10`
    - Autonomous System Number (ASN): `65009` (local) peering to `65000` (remote)
    - Router ID: `172.27.2.1`
    - Advertise Network(s): `172.27.2.0/24`

!!! "example" Configuration Parameters for VPN-based peers
    - Interface for routing link: `vpn0`
    - Tunnel IP's: `10.99.0.3` (local) to `10.99.0.2` (remote)
    - Autonomous System Number (ASN): `65009` (local) peering to `65000` (remote)
    - Router ID: `172.27.2.1`
    - Advertise Network(s): `172.27.2.0/24`


## Preparing Interfaces for the "Unnumbered" BGP Configuration

!!! "attention" Ethernet Only
    This section does not apply to routing configuration over WireGuard VPN links

To begin, we need to prepare the interface that we'll be using to peer with neighboring routers. In the past, this would have meant configuring IPv4 addresses for each side of the connection; however, we will be taking advantage of _unnumbered BGP_ to identify and connect to our peers based on auto-configured IPv6 addresses. This simplification is welcome since the addresses we assign on point-to-point routing links generally serve no purpose other than facilitating communication between the adjacent routers.

Our main task is to enable auto-configuration for IPv6 and to disable auto-configuration privacy extensions so that our MAC address will be encoded into the _link-local_ IPv6 address for the interface. We also need to ensure that the routing interface will listen for and accept _ICMPv6 router advertisements_ from its peer.

### Configure interface 
Create or edit a networkd `.network` configuration for the routing link, e.g., `/etc/systemd/network/24-vlan10.network` and ensure that the following IPv6 settings are in place. No IPv4 configuration will be performed on this link.

```
[Match]
Name=vlan10

[Network]
# Enable link local addressing for IPv6 only
LinkLocalAddressing=ipv6
# Watch for router advertisements so that we can learn about neighboring routers
IPv6AcceptRA=yes
# Required due to a bug in FRR (https://github.com/FRRouting/frr/issues/2205)
IPv6PrivacyExtensions=no
```

## Configure FRRouting
###  Set up address for BGP router ID 
The router can use any locally defined address as a router ID attached to BGP advertisements. Though we could use any one of our directly attached addresses, it is common to configure an ID on the loopback interface of the network device to ensure that it is available regardless of link status -- static addresses attached to physical interfaces are offline when the link is down.

We'll perform this configuration within the FRR VTY shell (_launch using `vtysh`_):

```
configure terminal
interface lo
ip address 172.27.2.128/32
quit    
```

### Additional "unnumbered" routing configuration

!!! "attention" Ethernet Only
    This section does not apply to routing configuration over WireGuard VPN links

Routers in an unnumbered peering relationship rely on ICMPv6 router advertisements to discover peers on a routing link. We previously configured our interface to accept these messages, but we still need to set up the routing daemon to generate the messages on the specified interface. 

The following commands will turn on ICMPv6 router advertisements and set an interval of one advertisement every 5 seconds (as recommended by FRR documentation).

Enter the following commands in the VTY shell:

```
configure terminal
interface vlan10
ipv6 nd ra-interval 5
no ipv6 nd suppress-ra
quit
```

### Configure BGP Routing
Once the preliminary configuration is complete, we can setup the BGP daemon to peer on the specified interface. The following examples provide the VTYSH commands needed to enable a new router in ASN 65009 to peer with a neighboring autonomous system and advertises routes for 172.27.2.0/24. Each example follows a common pattern:

1. Enter the configuration mode of VTYSH using `configure terminal`
2. Configure the BGP routing features by specifying your local ASN e.g., `router bgp 65009`
3. Set the router ID using `bgp router-id 172.27.2.128`. 
    - We set this address up earlier as a loopback since it will be seen on all routing links
4. Issue one or more `neighbor` statements, specifying either the interface (for unnumbered links) or the remote peer's address for other links.
5. Issue one or more `network` statements, enabling FRR to advertise routes associated with the given prefix.

#### Configuring an "unnumbered" routing peer over Ethernet

```
configure terminal
router bgp 65009
bgp router-id 172.27.2.128
neighbor vlan10 interface remote-as external
network 172.27.2.0/24
quit
```

#### Configuring a routing peer based on layer-3 adddresses (works for WireGuard)
```
configure terminal
router bgp 65009
bgp router-id 172.27.2.128
neighbor 10.99.0.2 remote-as 65000
network 172.27.2.0/24
quit
```

## Finalizing your Configuration

### Save Your Changes
Don't forget that live updates to FRR are not persistent. Rebooting the router or restarting `frr` will dispose of any settings that are not written to the startup config.

To save your changes, you need to be in _enable_ mode. Exit out of _configure_ mode, if necessary, and call `copy running-config startup-config` or `write memory`.


### Making Further Updates
Supporting multiple routing links on a single router requires a simple modification to this script to add additional `neighbor` statements. If you've already configured other details, you don't need to repeat them to add an aditional neighbor. Enter the following commands to resume editing your existing router:

```
configure terminal
router bgp 65009
```

From the Linux commandline, you can view the current _startup configuration_ within `/etc/frr/frr.conf`.
