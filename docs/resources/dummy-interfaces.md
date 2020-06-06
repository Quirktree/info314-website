# Working with Dummy Interfaces in Networkd

Dummys are functionally identical to loopbacks [(read more)](/resources/loopback-interfaces), though we're allowed to create more than one. Like a loopback, a dummy interface does not require a physical connection, and the addresses associated with the interface won't go up and down based on link status. A dummy interface can also be configured with one or more IP addresses and other layer-3 settings -- just like a physical interface. Likewise, we can bind daemons and sockets to our loopback/dummy addresses and respond to network requests received at these addresses. 

We will use dummy interfaces to extend our network virtually for the purpose of labs and to provide logical separation between multiple services hosted on a single physical node.

## Creating the Interface

Networkd supports various software-defined interface types, including dummy interfaces. We can create new interfaces manually using the `ip link` command, but the results will not be persistent.

!!! example "Manually creating a dummy interface"
    ```
    ip link add dev dmz0 type dummy
    ip addr add dev dmz0 10.10.10.10/32
    ```

In order to configure a dummy interface persistently on a system running networkd, the link settings should be added into a `.netdev` file and loaded into the networkd configuration directory (typically /etc/systemd/network). In addition to providing a name for the interface, the parameter `Kind=dummy` will trigger networkd to add a dummy interface and make it visible to other standard networking tools. Layer 3 configuration for this interface is defined separately in a `.network` configuration file. 

!!! example "Creating and configuring a dummy interface persistently"
    __`/etc/systemd/network/10-dmz0.netdev`__
    ```
    [NetDev]
    Name=dmz0
    Kind=dummy
    ```

    __`/etc/systemd/network/10-dmz0.network`__
    ```
    [Match]
    Name=dmz0

    [Network]
    Address=10.10.10.10/32
    # Include an Address statement for each IP required
    ```
    


