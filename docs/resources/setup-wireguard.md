# Installing and Configuring WireGuard on Raspbian and Debian-based Linux

## Before you Begin
These instructions will aide you with installation of WireGuard on Raspbian/Debian Buster. They have been tested on the February 2020 release of Raspbian Buster and on a Debian 10.3 droplet on DigitalOcean. Instructions may differ for other distributions or releases.

## Installing WireGuard

### Locating an Installation Package

While WireGuard support has been added to recent versions of the Linux Kernel, this milestone was not reached until after the Debian and Raspbian teams had frozen features for Buster.

At the time this guide was written, the WireGuard application and kernel module were not yet available in the mainline Debian and Raspbian package repositories. However, we can obtain these components from Debian's backport repository for Buster, a collection of new and updated software compiled for Buster.

Depending on your installation, *buster-backports* may already be listed as a valid software source. We found this to be the case for Debian 10.3 on DigitalOcean, though on Raspbian, we needed to manually add *buster-backports* as a new source and import the signing keys used to verify the integrity of software installed from the repositories. 

#### Check Existing Package Sources

Before you proceed, determine whether manual configuration of backports is necessary:

```bash
# Search existing apt package source locations
apt-cache policy | grep buster-backports
```

#### Manually Configure Backports

After completing the following steps, *apt* operations will consider packages from the backports repository while still preferring stable software versions from the mainline repository when available.

```bash
# Import the PGP/GPG signing keys for buster-backports from a key server
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 04EE7237B7D453EC 648ACFD622F3D138

# Add the backports repositories to the apt configuration
echo "deb http://deb.debian.org/debian/ buster-backports main  contrib non-free" | sudo tee -a /etc/apt/sources.list.d/debian-backports.list

# Update the local software catalog via apt
sudo apt update
```

### Install Linux Kernel Headers

The WireGuard package in backports repository attempts to compile a kernel module in order to add WireGuard networking capabilities. To complete this task, the compiler must have access to kernel headers, a set of files that define the interface of kernel functions and data structures.

The kernel headers are available as a package and accessible via *apt*. Installation depends on your chosen distribution:

#### Installing Kernel Headers for Raspbian

```bash
sudo apt install raspberrypi-kernel-headers
```

#### Installing Kernel Headers for Debian

```bash
# `uname -r` is used to specify the proper kernel version
sudo apt install linux-headers-$(uname -r)
```

### Install WireGuard and the DKMS-based module

Finally, with the pre-requisites in place, we can use *apt* to install WireGuard and all of its supporting components. Once installed, you can use *modprobe* as shown below to verify that it the WireGuard kernel module is recognized by the kernel.

```bash
# Install
sudo apt install wireguard wireguard-dkms

# Verify (should return without any output/error)
sudo modprobe wireguard
```

## Getting Started with WireGuard

The aim of the WireGuard project is to provide a fast and simple VPN that relies on rock-solid, modern cryptography. 

### Generate a WireGuard Key Pair for each Server

Every node participating in a WireGuard VPN is required to have a private key for its own use and a public key that will be distributed to each of its peers. Before setting up our own VPN, we need to generate these keys on each server. 

The following commands open a root-privileged shell in order to generate the keys. This extra step allows us to ensure proper file ownership and permissions via *umask* (which doesn't work smoothly with sudo)

```bash
# Open a root shell
sudo bash
# Override default file creation permissions
umask 077
# Create the public and private keys in /etc/wireguard
wg genkey | tee -a /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
# Close the root shell
exit
```

After generating the keys for your servers, be sure to document the publickey for each endpoint. You’ll need it again in order to configure each of your VPN peers.

### Identify Additional Parameters for your VPN

#### Endpoint IP and Ports
The endpoint addresses of a WireGuard VPN are the external addresses that peers will use in order to initiate connections with one another and to forward the encrypted payloads (via UDP datagrams that fully encapsulate the traffic sent/received on a virtual network interface). Each endpoint of a WireGuard VPN can function as both client and server (hence the name peer), however, taking full advanatage of this capability requires that all endpoints have routeable addresses. 

Due to NAT and/or firewall configurations, we may have endpoints that aren't directly reachable. WireGuard provides utilities to cope with this scenario, as long as at least one endpoint is able to receive incoming sessions from its peer(s).

As with all servers, a routeable peer must define a port on which it will listen for incoming messages. WireGuard does not provide a standard port configuration, so it is up to administrators to determine an appropriate UDP port for WireGuard server to listen.

Since we will be configuring multiple instances of WireGuard on our servers, we should pay close attention to which ports are in use so that we don't cause a collision between independent tunnel interfaces.

In the following examples, we will assume that we have one publicly routeable peer and one located behind a NAT. We'll use port 51820 to listen for connections on the routeable peer.

#### Tunnel IP addresses

WireGuard VPN connections are presented at the OS level as layer-3 network interfaces and will be configured with their own IP subnet/address settings. The main constraint is that these addresses are not part of any other subnet connected to the same device. For example, if my LAN is using 172.27.2.0/26 and my home network is 10.0.1.0/24, I cannot use anything from 172.27.2.0 - 63 or 10.0.1.0 - 255. I could, however, use addresses above 172.27.2.64. 

The constraint above isn’t anything unique to WireGuard, we’re simply following the principle that you can’t route between networks with overlapping IP ranges (in the same way that we can’t give the same address to two nodes on the same network).

The following examples will define tunnel addresses of 10.99.0.2 and 10.99.0.3 for traffic sent inside the VPN.

### Create WireGuard Interfaces

In order to send and receive network traffic through a VPN, each peer is configured with a virtual network interface. Any traffic sent to this interface will be transported over the VPN tunnel. Incoming traffic will egress on this interface after being processed and decrypted by WireGuard.

The following examples demonstrate the use of the *ip* command to manually set up the virtual interface for WireGuard. You should be aware that any configuration you set up with this utility is not persistent. It will be reset after a reboot. Persistent WireGuard configuration will be covered separately.

```
# Create the virtual interface
sudo ip link add dev wg0 type wireguard

# Assign a tunnel address to the interface (your second peer will reverse the order of these arguments)
sudo ip addr add dev wg0 10.99.0.2 peer 10.99.0.3

# Bring the interface online
sudo ip link set up dev wg0
```

### Create your Peers
Configure wireguard to use the new virtual interface to maintain a tunnel with its peer. As in the previous step, we’ll use the CLI interface to do this manually and come back later to add a persistent configuration.

```
# Gather the public addresses for your peers (as applicable). You should have a public address for your DO peer, and you will need it to set up the Pi. Since the Pi is behind a NAT on your home network, we can't reach it directly with a public IPv4 address. As such, we'll always rely on the Pi to initiate the connection to Digital Ocean. I'll discuss implications of this later and how to deal with any limitations.

# The core command looks like `wg set <INTERFACE> listen-port <PORT> private-key <PRIVATE KEY PATH> peer <PUBLIC KEY OF PEER> allowed-ips <List of CIDR IP ranges> endpoint <PUBLIC IP OF PEER>:<PORT>`. 

# Raspberry Pi / no public IP to listen on (replace public key and address of DO endpoint)
sudo wg set wg0 private-key /etc/wireguard/privatekey peer ABCDEFG allowed-ips 0.0.0.0/0 endpoint A.B.C.D:51280

# DigitalOcean / has its own public IP to listen on but can't reach RPi directly
wg set wg0 listen-port 51280 private-key /etc/wireguard/privatekey peer ABCDEFG allowed-ips 0.0.0.0/0
```

Observe how we adjusted the command when one of the peers doesn’t have a routable public IP. The downside of this scenario is that the tunnel can only be initiated from one side of the link, and we won’t be able to receive inbound traffic until it’s open. We’ll come back to discuss how to set up a keep-alive so that the tunnel is always open. When we have the luxury of public IP addresses for both peers, we should aim to set up both ends to listen and to connect to the peer endpoint.

### Test your Tunnel

If everything has gone well to this point, make sure that you can ping each peer using its tunnel IP address. This may not seem like the most impressive feat , but we’ll quickly expand the functionality by adding the ability to forward traffic between other attached networks. In order to do this, we need to set up routes for each subnet and add iptables rules that allow traffic to be forwarded on our new interfaces.

If you’d like to experiment with this on your own, check out the ip route command in order to program static routes. Our upcoming tasks will focus on using dynamic routing to automate the route configuration for the entire class network (which will have more than 30 routers and subnets).