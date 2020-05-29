# Final Network Project (last edited 2020-05-28)

## **Overview**
---
Working in your group, design and implement a networked system that mirrors the concept of an Internet Service Provider (ISP) with customer networks. Your primary objective in this exercise is to adapt what you've completed in previous tasks to meet the requirements specified below.

Although you will be working together as a group on this project, each participant will take the lead on configuring their own network device to integrate into the ISP network.

Additional configuration guidance will be made available through Slack. We will walk through new tasks together in our remaining lectures/labs.

!!! important "Important Instructions"
    As soon as possible, please browse to following link and provide the requested information about your group's WireGuard configuration and individual .pi Name Servers:

    - [PICANN Registry](https://docs.google.com/spreadsheets/d/1A3uCrVmrwoPdA0LvMH6IyRkU1_BLXhbVRjIadtc__E0/edit?usp=sharing)

    Take time to plan out the network design and IP address usage with your group before you build. This plan is a mandatory deliverable and will be required before implementation assistance is given.
    
    We recommend that you sketch out an initial diagram of the entire network and create a checklist of major settings for each device.

## **Before you start**
---

Each of your group members need to have completed the individual project checkpoints before proceeding with the configuration necessary for the final network. Each pi should function as a network gateway while also providing DHCP and private DNS services to an Ethernet-based LAN.

## **Project Details**
---

The following outline summarizes the steps that are needed to inter-network your individual LANs and join them up to the global, PiCANN internet core. Once you have completed these tasks, you will be able to route traffic across our class network and participate in friendly email exchanges with your classmates on the .pi top level domain!

1. With your group, <ins>review</ins> the ASN and address ranges defined at PiCANN and <ins>assign and ASN and distinct subnet</ins> to each group member.
1. With your group, <ins>configure</ins> a Debian 10.3 VM at DigitalOcean (or other cloud provider) to act as an <ins>ISP router</ins> for your group using VPN-based routing links.
1. <ins>Design a new address plan</ins> for your LAN/WAN presence based on the address range assigned in Step 1 and including a <ins>minimum of two subnets</ins>.
1. After reviewing your work on previous checkpoints, <ins>"re-number" your LAN segment</ins> using the new LAN subnet, updating the configuration of networkd, DHCP, iptables, and BIND as required for the task.
1. <ins>Configure a routing link</ins> between your Pi-based router and the cloud-based ISP, setting up both ends of the VPN and the BGP peering relationships.
1. Review the documentation you created for your .pi domain in Checkpoint #5 and <ins>update your DNS plan</ins> according to your new address plan.
1. <ins>Configure additional "DMZ" addresses</ins> on your Pi and <ins>update your DNS zone configuration</ins> to serve the new addresses correctly.
1. <ins>Install and configure</ins> the Docker-based mail servers based on your own network settings. 
1. Work with your group to <ins>test and document</ins> your work, ensuring that routing and DNS are working consistently across your shared ISP and individual Edge routers.
1. Using your shiny new webmail service, send an email to clinton@gradebook.pi.

### **Planning**
---

Up until this point, you have each configured your networks with private address ranges of your own choosing. While this is convenient for getting started, it doesn't lend itself well to a class-wide internet where each device must have its own unique addresses. Going forward, we will ask you to re-number your networks to use the assigned address blocks. This will require a small amount of collaboration within your group to assign sub-ranges.

#### **Specifications**

1. Assign an ASN and a distinct subnet to each member of your team. All ASNs and subnets must fall within the larger range provided for your group.
1. Select an additional ASN to use for peering between your ISP router and the PiCANN core. Update the PiCANN peering directory with this information.
1. Document your team's address range and individual assignments within a markdown document called final-planning.md. This document should be uploaded to your team's repository as soon as possible for referenced during the remaining parts of the project.

### **Documentation**
---
As you work through each part of this project, carefully document the decisions that you make. The process of planning and recording documentation will help you to avoid mistakes and simplify the troubleshooting process. Likewise, you will be asked to provide a comprehensive view of your network design within your GitHub repository. Detailed documentation requirements are provided within the Canvas assignments.

### **ISP Router**
---

As a group, you will need to set up a new Debian 10.3 droplet at Digital Ocean, install WireGuard and Free Range Routing packages, and configure a WireGuard and BGP to create a routing link with the core network. This link will provide connectivity to the other groups in the class, and the peering relationship you create with the core router will be used to exchange routes between upstream and downstream routers. 

Completion of this task will require collaboration with the Instruction team. Consult the [PiCANN] registry for VPN/BGP parameters that your group will use to peer with the core router. Fill in the missing values with your public key and the ASN that you assigned to the ISP router. Notify the instruction team via Slack so that we can complete the upstream configuration.

#### **Specifications**

1. Create a new VM using the $5/mo specs and the SSH key of one of your group members. You may also wish to enable IPv6 (can improve connectivity if any of your team mmembers have IPv6 capable ISPs (such as Comcast).
1. Update /root/.ssh/authorized_keys on your VM to include public keys for each of your group members as well as the following key for the instructor: `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBJTZTstXqjM7lEaFj/pnRNEztiZYu4yhKBZDrUCYMi5 clinton@info314`.
1. Install WireGuard and Free Range Routing (FRR) based on the instructions given in class and the guides posted on this site.
1. Create a new private/public key pair and add the public key to PiCANN along with the public IPv4 address of your router.
1. Referencing your groups configuration parameters in PiCANN, configure a persistent VPN interface to connect to the core based on the _legacy Debian networking_ approach.
1. Configure FRR to enable BGP and establish a peer relationship with the PiCANN core.
1. Refer to the documentation from Checkpoint #3 to enable IPv4 forwarding with _sysctl_ and create _iptables_ rules to allow packets to be forwarded from your WireGuard tunnel interface. As with Checkpoint #3, set the default forwarding policy to drop packets.
1. Make sure that you have updated the peering worksheet in PiCANN with missing parameters and notify the Instruction team that your peer is ready to connect.

### **LAN/WAN Address Planning**
---

#### **Specifications**

- Each group member should subnet their PiCANN subnet into 2 - 4 distinct subnets.
    - A minimum of two subnets per LAN are needed to meet the base project requirements.
    - If you would like to experiment with more complex configurations later we recommend four subnets.
- Use one of your subnets for your internal LAN (providing DHCP and other services to devices accessing through Ethernet). A second subnet should be dedicated to any public-facing services. We'll often associate this subnet with your "DMZ". 

### **Edge Network Setup**
---

After you complete your final planning for your LAN and public address ranges, you should "re-number" your LAN connection to use the new LAN address range. To save yourself significant troubleshooting time, proceed carefully with this task and make sure that you apply updates to networkd, DHCP, iptables, and BIND as required to support the new addresses. It will likely be helpful to review your previous work and the guides for the past Checkpoints (especially 1 - 4). 

The next task in store for you is to establish connectivity and routing to the ISP VM that your group configured earlier. This is a two-part process, as you will need to create a WireGuard interface and configure peering on both routers (Debian VM and Pi). Keep in mind at this point that there are slight differences in the implementation on each end of this connection. The WireGuard configuration on the Debian VM relies on _legacy Debian networking_ configuration while the Pi will be configured with the built-in WireGuard capabilities of _systemd-networkd_. We have provided reference documentation and video demos for each scenario.

The tasks you are completing are not extremely difficult, but there are ample opportunities for confusion. You should review instructions and make sure that your planning documents include all of the IP addresses and other values that you will need. Having this information well organized can be the difference between finishing this task in an hour versus spending several hours debugging.

#### **Specifications**

1. Update all network-based services on your LAN to use the new IP address ranges as determined by your group assignments and your own address planning.
1. Follow the linked resources to install WireGuard and Free Range Routing (FRR) on your Pi-based router.
1. Create a new WireGuard interface and configure it to peer with the Debian-based ISP router shared with your group.
    - Configure both ends of the VPN tunnel with IP addresses selected from your public/DMZ subnet. These addresses should be documented on your report.
    - Since your Pi router is most likely located behind a NAT, it will not be possible for the ISP router to open/reopen the WireGuard tunnel for traffic directed to your Pi. Configure a _persistent keep-alive_ of 25 seconds in the WireGuard peer settings on the Pi.
1. Enable the FRR BGP daemon and configure your new router based on the provided guides. In order to do this, you will need an ASN and an additional public address to use as a router ID. Make sure that both values are included in your documentation.
    - Log into the ISP router and update its BGP router settings to recognize your Pi-based router as a neighbor You will need the tunnel addresses from the previous step and the ASN identifiers assigned by your group to the ISP and your Edge network.
    - Repeat the configuration process on your local router, setting up a peering (neighbor) relationship with the ISP.
    - Use the _network_ directive to configure your local BGP to advertise the entire LAN/WAN subnet range that you were assigned by your group.
1. Update iptables to enable packet forwarding from your new WireGuard interface.


### **.Pi DNS Resolution**
---

Add the following section to your /etc/bind/named.conf.local. If you've configured separate views for internal and external name resolution, this will be inserted into the internal view.

```
zone "pi" IN {
  type slave;
  masters { 10.10.10.10; };
};
```

### **Public Services**
---
Use systemd-networkd to create and configure a persistent _dummy_ interface on the Pi as described in the attached resources. This interface will represent the "demilitarized-zone" of your network within which you will host your public services. For each service, you will need to set up an IP from your public subnet within the DMZ.

Follow the instructions provided in the [Pi Mail](https://github.com/i314-campbell-sp19/docker-mail) repository to install Docker and configure the SMTP, IMAP, and Rainloop containers to serve email for your domain. Be sure to map each of these servers to a public IP address within your DMZ.

Set up your Authoritatitive Name Server to listen on a public IP address within your DMZ. Update all records, e.g., MX and webmail, to point to the correct IP addresses based on your final network planning.


### **Making Improvements and Extra Credit**
---
- Add an Anycast route to 10.10.10.10 and host a copy of the .pi top-level-domain
- Configure a private DNS zone, e.g., corp.gradebook.pi, and use BIND views to restrict access to your private zone and the DNS resolver so that it is not available outside of your local LAN.
- Configure policy-based routing to send all of your DNS traffic through the VPN (may be necessary if your ISP is messing with your DNS).
- **Carefully** perform hardening of your firewall rules to provide stronger security for your LAN and your router.
- Add a wireless access point to your Edge LANs. The wlan0 port is capable of operating as a software-based access point using the hostapd package in the Debian repositories.

#### **Firewall Rules**

**Carefully** perform hardening of your firewall rules to provide stronger security for your LAN and your router.

Use the forward chain to prevent new connections into your LAN subnet from the routing link. In order for your users to access the rest of the Internet from your, you must still allow related/established connections back into your LAN.

**Carefully** extend your Iptables rules to support a default DROP policy on the INPUT and OUTPUT chains, making sure that you have full utility of all of your private and public services. When experimenting with default policies on the INPUT or OUTPUT chain, you should _ALWAYS_ use the _iptables-apply_ command to test your rules and confirm that you are still able to create new SSH sessions. Make sure that you have written appropriate rules to accept traffic for authorized services, such as DHCP, SSH, and DNS, before changing the policy.

### **Links and Resources**
---

- [PICANN Registry](https://docs.google.com/spreadsheets/d/1A3uCrVmrwoPdA0LvMH6IyRkU1_BLXhbVRjIadtc__E0/edit?usp=sharing)
- [Pi Mail](https://github.com/i314-campbell-sp19/docker-mail): Instructions and docker components needed to configure SMTP, IMAP, and a webmail server on your Pi
- [Installing WireGuard](/resources/setup-wireguard)
- [Installing Free Range Routing (FRR)](/resources/setup-frr)
- [WireGuard Peering](/resources/wireguard-configuration)
- [Configure Routing Links](/resources/configure-routing-links)
- [Configure Dummy Interfaces](/resources/dummy-interfaces)


<!--
Each group is required to fulfill the following requirements by building off of previous checkpoints and additional resources.

!!! important "Differences between videos/specifications"
    As a resource, I have provided videos that walk through various parts of this project. Please note that some of these videos were made in previous quarters. There may be minor differences between current project requirements and what is discussed in the video. It is important that you follow the current written specifications.


Perform IP address planning and network renumbering on a group and individual basis based on the address ranges provided by the instructor. 

  - IP address
      -  assignments can be found in the [PICANN Registry](https://docs.google.com/spreadsheets/d/1Bxp3wKK4W0cgG3RDtyWJg1VPGIyQEE0qn2TV0Y84Gx0/edit?usp=sharing)
      - Video guide on Authoritative server, external IP, external DNS: [Panapto Recording](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=65c5c73b-5d96-455b-8ee9-aa5e001e40c3)
      - [Address Planning for Final Project (video)](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=07ea5431-49f2-4022-b209-ab7001454435)
  - VLANS:
      - [VLAN and Switch Planning (video)](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=abcfc06f-b904-4cfc-8de9-aa64000aa3a0)
      - Planning: VLAN and switch planning worksheet
  - **Your network plans must be documented before requesting implementation assistance from instruction team**

Once you've completed all network planning and documentation you may move on to the following sections.

- Build a “core router” that will function as the ISP for your other networks. In addition to providing connectivity between routers, you will provide public Internet access for the rest of your group. The router will eventually be connected to other groups to form a private in-class Internet.
    - [Project Topology (video)](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=fc325b42-6bed-4542-8940-ab700140b0c3)
    - [VLAN and Switch Planning (video)](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=abcfc06f-b904-4cfc-8de9-aa64000aa3a0)
- Build two or more “edge routers” that will allow your partners to connect to the Internet as customers of the ISP. Each router should serve one or more LAN subnets. In your final configuration, you should rely on the ISP connection for public Internet access, i.e., you won't be connecting the edge routers to public wifi directly except for setup purposes.
- Using Free Range Routing (FRR), configure BGP peering on the routing links between the ISP routers and customers’ edge routers.
    - [Setup FRRouting](/resources/setup-frr)
    - [Install FRR and Unifi Controller (video)](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=abd934e0-9a2b-4c4e-9e76-aa63017e78f0)
    - [Configure BGP peering](/resources/configure-routing-links)
    - [FRR and BGP routing configuration (video)](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=b55f0a5f-4dc4-4fee-b8ee-aa64001a1d50)
- Configure public domains, authoritative servers, and (optionally) email for each of your networks. One router in your group should also host the .pi TLD at 10.10.10.10 (BGP Anycast) via a provided Docker image.
- **\[ SKIP ME \]** Configure a Ubiquiti Unifi switch based on instructor instructions to support the system topology.
    - [Setting up VLANs on a Unifi Switch (video)](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=daee7090-819a-4768-83dd-aa630185d228)

### Renumbering IP Addresses
- A block of IP addresses have been assigned to each group via the [PICANN Registry](https://docs.google.com/spreadsheets/d/1Bxp3wKK4W0cgG3RDtyWJg1VPGIyQEE0qn2TV0Y84Gx0/edit?usp=sharing)
- Assign a range of addresses to each group member based on this block.
- Each group member should further subnet their own range into 2 - 4 distinct subnets.
    - A minimum of two subnets per LAN are needed to meet the base project requirements.
    - If you would like to experiment with more complex firewall rules and/or setting up a wireless LAN, we recommend four subnets.
- Use one of your subnets for your internal LAN (providing DHCP and other services to devices accessing through Ethernet). A second subnet should be dedicated to any public-facing services. 

### Core Router Setup
- Managing network interfaces
    - Use wlan0 to provide access to the public Internet to your customers (NAT is required)
    - **\[ORIGINAL\]** Set up tagged VLAN interfaces to connect with each edge router.
    - **\[UPDATED\]** Set up at least one VLAN interace, as would be required to connect with one of the Edge routers included in your network plan.
    - See [Setting up VLAN interfaces in Linux (video)](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=984e3939-da15-414a-9af4-aa640010989e).
- Firewall rules
    - Our network is private, so use connection state with your forwarding rules to make sure inbound packets (from wlan0) are dropped when they aren't related to established sessions
    - Within our network, traffic should be routed rather freely. In other words, make sure you aren't dropping packets that are being forwarded from your LAN or one of your partners' routing links.
- Private LAN
    - Even though your router's primary role is infrastructure for your group, you should still set up a basic LAN that you'll connect to for management purposes. This should meet the base requirements described for LAN services.

### Edge Network Setup
- Managing network interfaces
    - Create a tagged VLAN to connect with the core router. You must coordinate the VLAN ID with the core router and the switch.
    - See [Setting up VLAN interfaces in Linux (video)](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=984e3939-da15-414a-9af4-aa640010989e).
- Managing firewall rules
    - You'll have no need for NAT as long as you set up your LAN inside your allocated IP range (see the previously linked registry)
    - You may set the default FORWARD policy to ACCEPT to ensure that traffic can be routed between partners.

!!! info "Extra Credit"
    - Configure the default FORWARD policy to DROP traffic.
    - You can allow traffic to be forwarded into your network (from wlan0) if it's related to a session your LAN client's initiated
    - You can allow traffic into your device or network from the other routing links if it is destined to a public-facing service (i.e., Authoritative DNS, SMTP, or Webmail).
    - Configure DROP policies for the INPUT and OUTPUT chains that manage traffic to/from the router.

### Ubiquiti Switch

#### **\[ SKIP THIS SECTION \]**

See [VLAN and Switch Planning](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=abcfc06f-b904-4cfc-8de9-aa64000aa3a0), [Installing FRR and Ubiquiti Controller](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=abd934e0-9a2b-4c4e-9e76-aa63017e78f0), and [Configuring Unifi Switch](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=daee7090-819a-4768-83dd-aa630185d228).

- Configure a VLAN that each group member shares with their own Pi. Set the VLAN up as native (untagged) for both the Pi and the PC connection to the switch.
- Set up tagged VLANs for each routing link. Create a profile for the port associated with each of your Pi’s and add the VLANs to the profile according to which connections are available to that Pi.
- Reserve one port for the connection to the “core network”. This port should be natively associated with the VLAN created for the core routing link.

### LAN Services
- Configure a DNS resolver (either caching or forwarding) to handle requests outside of your private zone. This was the outcome of Checkpoint &num;4.
- Configure DHCP services to distribute addresses and configuration within your LAN to end-users ([use the pre-defined address allocations for your group]). This was the outcome of Checkpoint &num;2, though you will need to adjust your address range.
- **(EXTRA CREDIT OPTION)** Configure a private DNS zone, e.g., corp.gradebook.pi, and use BIND views to restrict access to your private zone and the DNS resolver so that it is not available outside of your local LAN.

### Public Services
See [Configuring External IP and DNS](https://uw.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=65c5c73b-5d96-455b-8ee9-aa5e001e40c3).

- Configure a public DNS zone for each network (ISP + customer networks) and set up records for each of your public services (you may choose to centralize authoritative DNS at the ISP, but each student must configure their own zone file).
- **\[ SKIP ME \]** (ONE PER GROUP) Host an authoritative DNS slave for the .pi TLD. This server will be hosted publicly on 10.10.10.10 and routed via BGP Anycast. A dockerized container and instructions to deploy are included in the repo linked below.
- **(EXTRA CREDIT OPTION)** Configure [email](https://github.com/i314-campbell-sp19/docker-mail) for each of your public domains using the provided docker containers.


## **Extra Credit**
---

If you're up for the challenge, there are several other ways to earn extra credit toward your final grade by completing the following tasks.

- **(1 pt)** Install and configure the [Pi Mail](https://github.com/i314-campbell-sp19/docker-mail) Docker containers in order to host email on your domain/network.
    - Extra credit will be given to each student who successfully completes this task.
- **(1 pt)** Add a wireless access point to your Edge LANs. The wlan0 port is capable of operating as a software-based access point using the hostapd package in the Debian repositories.
    - At least two group members must complete this configuration. Instructions will be provided.
- **(2 pt)** Extend your Iptables rules to support a default DROP policy on the INPUT chain, making sure that you have full utility of all of your private and public services.
    - Apply this configuration to each of your customer networks, adapting for any differences in configuration for that network.
- **(1 pt)** Extend your IPTables rules to support a default DROP policy on the OUTPUT chain, making sure that you have full utility of all of your private and public services.
    - Apply this configuration to each of your customer networks, adapting for any differences in configuration for that network.
- **\[ SKIP ME \]** (1 pt) Perform TCP-based nmap scans of your networks from inside and outside. Summarize the major findings for each network.
    - You should have 2 scans per Pi. One scanning IP addresses for that team member from inside their LAN and one scanning from a different LAN.


## **Important Links**
---

- [PICANN](https://docs.google.com/spreadsheets/d/1Bxp3wKK4W0cgG3RDtyWJg1VPGIyQEE0qn2TV0Y84Gx0/edit?usp=sharing): DNS Registry and IP Reservations
- **\[ SKIP ME \]** dotPI TLD:  Instructions and docker components needed to load a dockerized BIND on 10.10.10.10 for the .pi TLD
- [Pi Mail](https://github.com/i314-campbell-sp19/docker-mail): Instructions and docker components needed to configure SMTP, IMAP, and a webmail server on your Pi
-->