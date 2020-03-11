# **Final Network Project** (last edited 2020-03-11)

## **Overview**
Working in your group, design and implement a networked system that mirrors the concept of an Internet Service Provider (ISP) with customer networks. Your primary objective in this exercise is to adapt what you've completed in previous tasks to meet the requirements specified below.

Although you will be working together as a group on this project, each participant will take the lead on configuring their own network device to integrate into the ISP network.

Additional configuration guidance will be made available through Slack. We will walk through new tasks together in our remaining lectures/labs.

!!! important "Important Instructions"
    Before you get started, please browse to following link and fill out IP and DNS info:

    - [PICANN Registry](https://docs.google.com/spreadsheets/d/1Bxp3wKK4W0cgG3RDtyWJg1VPGIyQEE0qn2TV0Y84Gx0/edit?usp=sharing)

    Take time to plan out the network design and IP address usage with your group before you build. This plan is a mandatory deliverable and will be required before implementation assistance is given.
    
    We recommend that you sketch out an initial diagram of the entire network and create a checklist of major settings for each device.

## **Before you start**
Each of your group members need to have completed the individual project checkpoints before proceeding with the configuration necessary for the final network. Each pi should function as a network gateway while also providing DHCP and private DNS services to an Ethernet-based LAN.

## **Project Details**
---

### Overview
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
