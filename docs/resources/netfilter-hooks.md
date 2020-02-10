#  Understanding Netfilter/Iptables Hooks
The _netfilter_ component of Linux defines a set of _hooks_ within the TCP/IP stack that are used to run code based on different packet events at different moments in the packet lifecycle. Utilities like iptables (and more recently nftables) provide a frontend and language for managing rules and actions that we want to perform based on the characteristics of network traffic. These tools are commonly used to implement services like NAT or to provide host or network-based packet filtering firewalls.

## Working with Local Traffic
To better understand how all of this works, let’s start with a simplified example on a standard Linux host. For now, ignore the possibility of routing and focus on the host’s ability to send or receive packets. In this scenario, we can use netfilter for tasks like dropping unwanted traffic or rewriting addresses and layer-3 ports on the fly.

Since this is a simplified scenario, we can limit our examination to two hooks, INPUT and OUTPUT, that are triggered for packets flowing to and from the local device in a non-routing scenario. By non-routing, we mean that the INPUT hook is only invoked when the host is the intended recipient of the message based on the packet’s destination IP address. Likewise, the OUTPUT hook won’t fire unless the message originated from the local host. Given the INPUT/OUTPUT hooks, we can use a frontend like iptables to register specific rules and actions that are evaluated for each packet passing through the network stack.

### Host-based Packet Filters
One of the most common applications of the INPUT/OUTPUT hooks is to perform packet filtering in order to provide a host-based firewall for the Linux device. For example, from a security perspective, we typically want to limit how much of an “attack surface” we expose over the network. We can leverage the INPUT hook to drop packets that aren’t part of a TCP connection that we had previously initiated. In other words, we allow client-based software to open outbound connections but we block any attempts to connect to us as a server. If we need more flexibility, we can create rules to allow connections in to specific services, such as an HTTP or SSH daemon. 

## Working with Routed Traffic
Netfilter operations become slightly more complex when routing is introduced. Like an ordinary host, routers still process packets coming to/from the local device, but they also need to forward packets received on one network interface through other interfaces. In fact, this latter operation is their main responsibility. In order to support these communication paths, netfilter defines three additional hooks: FORWARD, PREROUTING, and POSTROUTING that we can use to perform network-based packet filtering, network address translation (NAT), and other packet-level operations.

### Network-based Packet Filters
For packet-filtering purposes, the FORWARD hook is a direct analogue for the INPUT/OUTPUT hooks. As we’ve mentioned, INPUT/OUTPUT are not applied to routed packets. Instead, code registered on the FORWARD hook will be evaluated after the initial routing decision is made. The FORWARD hook provides the basis for network-based firewall functionality. Network-based firewalls enable us to drop or reject unwanted traffic before it has the opportunity to reach our hosts (or even enter into the network). Likewise, we can also leverage packet filtering to prevent certain types of local traffic from inadvertently leaking outside our own network perimeters.

### Network Address Translation
Sometimes it makes more sense to perform an action for an incoming packet before any routing decision has been made or just before actually sending an outgoing packet. This is the case for port and network address translation and is facilitated by attaching rules to the PREROUTING and POSTROUTING hooks. Specifically, we set up rules on the PREROUTING hook when we want to modify incoming packets based on their destination IP address. This configuration is called destination NAT (DNAT) and is useful for delivering traffic to a server on a private subnet. Likewise, we use the POSTROUTING hook when our goal is to change outgoing packets based on the original source address. This configuration is called source NAT (SNAT) and is useful when we need map a public IP on to packets coming from private addresses. 

### Additional Caveats for Local Traffic
Unlike the FORWARD hook, PREROUTING and POSTROUTING may be used for local traffic as well as routed traffic. In particular, an incoming packet will be processed by the PREROUTING hook before the network stack makes a routing decision and reaches the INPUT hook. For outgoing packets, the OUTPUT hook is invoked prior to the outbound routing/forwarding decision or the POSTROUTING hook. For local traffic, we often have a choice about which hook provides the best entry point for our rules. 

## Summary
When you're first learning about Linux networking internals, it can be difficult to remember the relationship between netfilter hooks and related events in the packet lifecycle. The following list summarizes what we've learned so far in terms of specific traffic flows. 

- Incoming local traffic: (receive packet via network interface) -> PREROUTING -> (local routing decision) -> INPUT -> (continue processing packet and deliver via sockets)
- Outgoing local traffic: (receive packet via sockets) -> OUTPUT -> (outbound routing) -> POSTROUTING -> (continue processing and send via network interface)
- Routed traffic: (receive packet via network interface) -> PREROUTING -> (inbound routing) -> FORWARD -> (outbound routing) -> POSTROUTING -> (continue processing and send via network interface)

As you work to digest each of these traffic flows, you can refer to the following list for a reminder of which operations can be performed by using a particular hook. Once again, we are distinguishing routed traffic from incoming and outgoing routed traffic.

- Filtering incoming traffic to local: INPUT
- Filtering outgoing traffic from local: OUTPUT
- Filtering routed traffic (in any direction): FORWARD
- SNAT for routed traffic: POSTROUTING
- SNAT for outgoing local traffic: POSTROUTING or OUTPUT
- DNAT for routed traffic: PREROUTING
- DNAT for incoming local traffic: PREROUTING or INPUT
