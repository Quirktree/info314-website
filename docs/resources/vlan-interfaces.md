# Configuring VLANs in Networkd

VLAN functionality in networkd enables Linux hosts to participate in VLANs based on explicit tagging of ethernet frames. VLANs provide a valuable layer of logical configurability on top of physical network infrastructure. VLAN connections are presented in Linux as software-defined network interfaces that can be configured and utilized just like other types of network interfaces.

## Creating VLAN Interfaces

Networkd supports various software-defined interface types that are created based on `.netdev` files in the networkd configuration directories. A `.netdev` for a VLAN sets `Kind=vlan` and defines the numeric ID used to tag VLAN traffic on the wire in order to keep it separate from other traffic. 

!!! example "Creating a VLAN interface"
    __`/etc/systemd/network/15-vlan25.netdev`__

    ```
    [NetDev]
    Name=vlan25
    Kind=vlan

    [VLAN]
    # Specify the tag
    Id=25
    ```

## Add VLANs to a Physical Interface

In order to send and receive tagged traffic over a network link, we need to associate the newly created VLAN interfaces with the physical LAN interface. In networkd, this can be done by adding one or more VLAN properties to the physical interface's `.network` configuration. After doing this, tagged frames will be presented to the operating system as originating from the associated VLAN interface. Likewise, traffic that is sent through a VLAN interface will be tagged with the specified ID before being sent on the network.

!!! example "Linking a VLAN interface to a physical interface"
    __`/etc/systemd/network/20-eth0.network`__
    ```
    [Match]
    Name=eth0

    [Network]
    # Existing Configuration
    Address=192.168.0.1/24

    # Tagged VLANs Referenced by Interface Name
    VLAN=vlan25
    VLAN=vlan50
    ```

## Network Configuration for VLAN Interfaces

Attaching the VLANs to a physical interface enables traffic to be sent and received from the virtual interface via tagged frames on the physical link. At the moment, however, our VLAN interfaces do not have any network-level settings attached to them.

Depending on the task at hand and the configuration of the Linux host, you may configure the new interfaces by any method(s) available, including networkd:

!!! example "Sample VLAN interface configuration"
    
    __`/etc/systemd/network/25-vlan25.network`__

    ``` tab="DHCP"
    [Match]
    Name=vlan25

    [Network]
    DHCP=ipv4
    ```

    ``` tab="Static IP"
    [Match]
    Name=vlan25

    [Network]
    Address=172.16.0.1/24
    DNS=1.1.1.1
    ```