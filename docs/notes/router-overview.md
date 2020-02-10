# Routers
Routers and routing are frequent topics when we discuss networking, and it is all too easy for newcomers to overlook or misunderstand some of the assumptions we make about these terms. Let’s take a step back and review the fundamentals.

## What is a Router
A router is a computing device that is attached to two or more networks _and_ is set up so that it can forward packets from one network to another. Both of these conditions are necessary in order for us to call something a router. You might think that all routers are special purpose, embedded devices like the ones you see on most home or office networks. In reality, most computers[^hotspots] are capable of routing, so long as the hardware supports multiple network connections and the software supports forwarding between networks.

[^hotspots]: When we say most computers, we mean it. Have you ever used your mobile phone as a hotspot for your tablet or computer? Give it a shot, and if you’re able, use `traceroute` or `tracert` to observe the routing process in action.

## Routers vs Multi-homed Hosts
It is not uncommon for a device to be connected to multiple networks without being configured to route traffic. A device with this configuration, e.g., your Pi prior to checkpoint #3, is known as a multi-homed host. A non-routing, multi-homed hosts will drop packets that aren’t directly addressed to it at both Layer-2 (MAC address) and Layer-3 (IP address). This behavior is exactly the same as an ordinary host attached to a single network.

## Routers and Incoming Packets
On an ethernet-based network, a router will process any frame for which the destination MAC address matches the MAC of the NIC on which the frame was received. After processing Layer-2, the router examines the packet’s IP headers. Assuming that the destination IP address is not a match for the router’s IP address, the router will consult its route table in order to forward the packet closer to its final destination. Packets addressed directly to the router will continue to be processed so that they can be delivered to an internal process via a socket. 

## Routers and Outgoing Packets
Most of the unique magic that distinguishes routers from ordinary hosts takes place on inbound packets. You might be surprised to learn that the process of sending a packet is the same for packets from local applications as it is for packets forwarded between networks. 

No matter where a packet comes from or where it is going, packet-communication occurs on a hop-by-hop basis. Every host starts this process by comparing its destination address to entries defined in its own routing table. Matches are identified when the network bits of the destination match a network id in the based on the correspondence netmask or CIDR defined in the routing table[^longest-match].

[^longest-match]: It is possible for a routing table to find multiple matches. In this case, ties are broken by looking for the most specific entry, i.e., the longest matching network prefix. Other metrics may be used if there are multiple entries for a given network ID.

After finding a match, the network stack can start the process of forwarding to the next hop. If the final destination is on a directly attached network, ARP is used to determine the MAC address of the destination and the packet will be forwarded directly. Otherwise, the routing table can be used to find a _next hop_ IP address for a directly attached network. Once again, with the next hop IP, ARP can be used to determine the MAC address so that the packet can be forwarded directly.

## Terminology
Routing terminology is a common sticking point for many students and professionals. In particular, it can be useful to understand the finer distinctions between the terms _routing_ and _forwarding_. At Layer-3, the word forwarding describes the mechanics of passing an inbound packet from its origin interface back onto a different network through an outbound interface. The mechanics of finding a matching route and pushing the packet out the appropriate interface are all part of the forwarding process. In contrast, routing refers to the entire end-to-end process that gets a packet to the right place.