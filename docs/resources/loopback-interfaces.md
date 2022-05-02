# Working with Loopbacks in networkd

When we have worked with IP addresses, we have mostly though about them as being configured in association with a particular network interface on a device. One challenge with this configuration is that addresses attached to a physical network interface will not be brought up if the link is unavailable. In certain use cases, we will want an address that operates independently of the physical links. A common application of this configuration, for example, is dynamic routing. 

Two of the mechanisms that Linux provides in service of this goal are loopbacks and dummy interfaces [(read more)](/resources/dummy-interfaces). The focus of this guide is on configuration of the loopback interface.

# Configuring additional loopback addresses
Linux allows exactly one loopback interface called `lo` to be allocated on a given system. You should already be familiar with this interface as the home of the locally scoped 127.0.0.1 localhost address. While we can't create a new interface, we are permitted to add new addresses to the existing loopback. Unlike the localhost address, these addresses will be globally scoped and remotely accessible.

Addresses can be configued using the `ip address` command for a manual, non-persistent configuration or via systemd-networkd `.network` files.

!!! example "Manually configuring secondary loopback addresses"
    ```
    ip addr add dev lo 10.10.10.10/32
    ```

!!! example "Configuring a persistent loopback address"
    __`/etc/systemd/network/10-lo.network`__
    ```
    [Match]
    Name=lo

    # Each address is placed in a [Address] section
    [Address]
    Address=10.10.10.10/32
    ```

## Resources
* https://manpages.debian.org/buster/systemd/systemd.network.5.en.html
* https://wiki.archlinux.org/index.php/systemd-networkd
