http://hackaday.com/2013/02/15/cracking-open-a-24-port-switch-so-you-dont-have-to/
http://blog.thelifeofkenneth.com/2013/02/tear-down-of-hp-procurve-2824-ethernet.html

Key concepts: Shared transmission medium, topology, hub, congestion, collision domain, broadcast domain, segment (layer-2), bridge, switch, forwarding information base (FIB)

# Introduction
Ethernet switches are one of the most widely deployed network components, providing physical ports for attaching other network components or endpoints at the data-link layer of the TCP/IP stack. Barring advanced configurations, devices that are attached through switching infrastructure are part of the same layer-2 network and can communicate directy through the switch using layer-2 addresses.

# Collisions and Media Access Control
Switches are not the only layer-2 game in town for wired Ethernet, though they are one of the few you should even consider in 2020. Once upon a time we connected devices in a *bus* configuration, where each node would tap into a segment of coaxial cable or other suitable transmission medium. In this configuration, the bus provides a resource that must be shared between all nodes attached to it. One consequence of this configuration is that data frames are visible to every other node since the signal is transmitted across the entire bus. Likewise, since the bus is shared, *collisions* between data frames may occur when nodes try to use the resource at the same time. For this reason, we say that bus represents a single *collision domain*. 

Ethernet copes with collisions using a *media access control (MAC)* mechanism that allows the NIC to identify a failure and make another attempt after a variable delay. This mechanism is known as *Carrier Sense Multiple Access with Collision Detection (CSMA/CD)*. While CSMA/CD enables the network to continue functioning despite collisions, *congestion* occur as the shared medium becomes more contested. A highly congested network is inefficient since failures and retransmission consume an increasing amount of its bandwidth.

# Star Topology
Congestion is not the only architectural concern associated with the bus topology. Physical challenges such as signal interference and attenuation limit the overall length of a bus. As such, the layout of the bus relative to the physical location of each node must be carefully planned. Hubs alleviated these latter concerns by offering a more flexible topology. In a *star topology*, multiple nodes connect back to a central point through dedicated, indepenent links, e.g., lengths of CAT-5 cable. 

Beneath the hood, hubs still operate on the underlying principle of transmitting signals across an entire network segment. In fact, each port of a hub acts as a repeater for the physical signals that were received on other ports. As such, hubs do not eliminate the concern of congestion and collisions or snooping on communication within a segment since endpoints connected by hubs are still members of a single collision domain. 

# Bridging and Switches
*Bridging* provide a layer-2 solution to the congestion problem by splitting a network into independent collision domains and intelligently forwarding frames between these layer-2 *segments*. When a bridge observes a frame on one of its ports, it inspects Ethernet headers to *learn* about the topology of the network based on the MAC address of the sender. The bridge uses the information it learns about the topology to optimize forwarding, which is only necessary when the recipient and sender are on different segments.

xxx NO CHANGES TO ETHERNET

Classic Ethernet bridges provides two ports/collision domains and are rarely seen in modern networks. A switch combines multiple bridges in a device that physically resembles a hub. Unlike a hub, however, each switch port represents a distinct segment on its own collision domain. From a performance perspective, a well-placed bridge can reduce the amount of activity on a domain by about half (discounting for broadcast traffic). Adding more bridges further improves the situation. In a switched network, every link is isolated within an independent collision domain, allowing a much greater level of overall throughput on the network.

## Forwarding Mechanics
The forwarding mechanics of a switch are identical to a bridge. As switches learn the topology of a layer-2 network, they populate a Forwarding Information Base (FIB) with MAC/port associations. Until a switch knows which port is associated with a particular destination MAC address, a frame received on one port will be *flooded* over every other port on the switch.


[^fib]: FIBs sometimes referred to as a MAC address table or a CAM table. CAM stands for content addressable memory, which is the implementation mechanism used for the FIB on many switches.

# Broadcast Domains
As we've seen, switches support standard Ethernet modes of communication between all nodes of the layer-2 network based on MAC address. 

# Virtual LAN (VLAN)

VLANs provide a solution to isolate broadcast domains from one another without requiring an investment of dedicated hardware for each domain. In the simplest sense, VLANs provide virtual separation of broadcast domains by identifying switch ports as VLAN members and referencing this configuration to enforce the separation of ethernet traffic.

A switch configured with more than one VLAN behaves like multiple standalone switches. The ports assigned to a given VLAN maintain their own view of the FIB that is based only on the MAC addresses seen on those ports. In port-based VLANs, which use the mode of operation described above, a router would need to be attached to each set of ports to assign forwarding across the VLAN boundary.

Many switches, support a more dynamic mode that enables network-attached devices to distinguish between traffic from multiple VLANs running over a single link. Links that carry traffic for multiple VLANs are called trunks (the associated switch ports are called trunk ports). With trunking, connections between infrastructure components is simplified since multiple broadcast domains can be collapsed onto a single physical link. Trunks generally rely on tags (as specified in 802.1q) to control distribution of frames over VLANs. 

Despite implementation differences, tag and port-based VLANs obey the same rules. Layer-2 frames received on any port/tag are only distributed within the associated VLAN.



# Switch Architecture Planning
Larger networks require multi-tier switch deployments to satisfy the requirements of core communication, distribution, and network access. Switches at the access tier need to provide enough port density to support endpoints in the network. Switches participating in the distribution tier of the network architecture may prioritize VLAN trunking, basic redundancy, and incorporate layer-3 functionality to handle routing and security policies internally. Core functionality will likely incorporate multiple levels of redundancy and higher capacity links.

# Redundancy and Switch Loops
Redundancy is a common requirement in network infrastructure for the corporate world. Organizations have a greater dependence on information technology and a low threshold for system outages that disrupt operations or productivity. In LAN design, we need to plan for redundant components and links based on the impact of a failure at a given point in the network.

Redundancy at this layer has an interesting effect with regard to forwarding behavior. A frame that passes through one side of a redundant connection is expected to return to the switch through the other side of the connection. This situation is commonly known as a switching loop. Frames that are caught in a loop degrade the performance of the switches involved. Broadcast traffic exacerbates the problem and may create a broadcast storm. Spanning tree protocol (STP) is present on many business-grade switches to alleviate this problem. 