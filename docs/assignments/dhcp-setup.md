# Set up a DHCP Server for your LAN (2020-01-22)

## Overview
In this assignment, we'll be standing up a DHCP service on the Raspberry Pi. As we've discussed in class, DHCP is a function that is sometimes performed by routers and is a necessary service whenever we are creating a new network (of almost any type). 

## Before you Start

Before starting any of the work within this checkpoint, make sure that you have completed all steps from checkpoint one, including the **networkd** setup.

You must also have completed the **Address Planning Exercise** correctly. You will use the parameters you selected in that exercise to configure the IP address for the Pi as well as the configuration for **isc-dhcp-server**.


## Configure Static Addresses for your Raspberry Pi

In this step, you will add a new configuration file to **networkd** in order to set up static addresses on **eth0**. The addresses you use here will be the one you defined for your **DHCP Server** and **Default Gateway** in the **LAN Planning Exercise**.

??? danger "Please do not try to follow other tutorials for setting up addresses"
    Every quarter, we have students who choose to follow online tutorials instead of the instructions we've provided. Please be aware that we are not using the default Raspberry Pi networking. These tutorials will often instruct you to configure static addresses in the `/etc/dhcpcd.conf` file or `/etc/network/interfaces`. Both of these methods were disabled when we set up **networkd**.

Create a file named `20-eth0.network` in  `/etc/systemd/network/` (you'll need to use `sudo`) and define a static configuration that includes your chosen IP address and CIDR range. This path should already contain a default configuration for ethernet interfaces, e.g., `99-eth.network`. We are overriding this configuration for `eth0` by naming the file in such a way that **networkd** will load it first and by ensuring that the configuration will match `eth0` exactly.

Your file should look something like:

```
[Match]
# We only want to match the eth0 interface
Name=eth0

# Provide full address config in abbreviated notation
[Network]
# Default Gateway / DHCP Server
Address=192.168.0.1/24


# Enable link local addresses for IPv6 (FE80::)
LinkLocalAddressing=ipv6
```

!!! warning Important 
    Some online references will have you add a gateway and DNS settings for your new interface. You don't need to (and definitely shouldn't) add either to this interface. 
    

    This interface does not provide a path for Internet traffic, but your Pi will try to use it that way if it believes it can be used for a default route.
    
    Likewise, we don't need to give the Pi DNS settings on Ethernet. The Pi will get it’s DNS from the wireless interface, which received it’s DNS settings from DHCP. 


In order to test these new settings:

Call `sudo systemctl restart systemd-networkd.service` **or** reboot your Pi.

Once you log back in, verify that `networkd` is running by calling `systemctl status systemd-networkd` and use the `ip addr` command to check that `eth0` is online with the new address.

!!! attention
    Make note that the Pi is now assigned to our new subnet, but our computer will still be using it's autoconfigured IPv4 address in the 169.254/16 range.

    In an IPv4 only world, this would prevent us from talking to the Pi. To fix the problem, we'd have to manually configure a static IP on our laptop in the same range as the Pi's network ID.
    
    Fortunately, both devices have autoconfigured IPv6 addresses that will be discovered through mDNS. By default, this is the address that SSH will use.

## Install and Configure DHCP
For this part of the exercise, we'll use the ISC DHCP Server. The ISC server is available in `apt` repositories, so it can be installed using:

```bash
sudo apt update
sudo apt install isc-dhcp-server
```

!!! warning Important
    Right after installation, isc-dhcp-server will report that it failed to run, and spit out lots of nasty-looking errors. This is normal, as the DHCP server has not yet been configured.

Edit `/etc/default/isc-dhcp-server` to specify the interfaces (`eth0` only) on which to run the DHCP server. Comment out any options pertaining to DHCP for IPv6.

The stock DHCP configuration is located at `/etc/dhcp/dhcpd.conf`. This file contains a number of example `subnet` declarations that you are meant to pick and choose from in order to configure the DHCP server and to distribute addresses within your predefined address range. For this assignment, our configuration is quite minimal:

!!! instructions "Specifications"
    Before you start to configure your subnet, comment out the lines near the top of the file that specify global settings for nameservers and domain (we won’t use them right now).

    * Create a new subnet configuration block using the network ID and network mask you established in the *LAN Planning Exercise*.
    * Inside the subnet block, define a contiguous `range` of IP addresses for the DHCP pool out of the address range selected in the *LAN Planning Exercise*.
    * You may browse the othe options available, but at this point you should not specify anything other than the DHCP lease range.

!!! faq "Why don't we configure the default gateway or name servers yet?"
    Students will frequently jump ahead and fill in additional network parameters. The reason we don't want to include these parameters yet is that our Pi is not yet ready to provide an Internet connection. Sending a default route back to your workstation will create a black hole of sorts for Internet traffic. Your computer will try to route the traffic through the Pi, and the Pi will simply drop everything it receives.

## Test DHCP Server
Use `systemctl` to restart the DHCP service as shown here:

`sudo systemctl restart isc-dhcp-server.service`

If you see errors such as the one shown here, you’ll need to continue troubleshooting the DHCP server (see next section).

```
pi@titan:~ $ sudo systemctl restart isc-dhcp-server.service
Job for isc-dhcp-server.service failed because the control process exited with error code.
See "systemctl status isc-dhcp-server.service" and "journalctl -xe" for details.
```

Check your local machine to confirm that an IP address in the specified network range was provided.

Try running `ping` from both directions (laptop -> pi and pi -> laptop) to confirm that the IP address and related settings are configured correctly.

!!! warning Important
    If you are on Windows, incoming `ping` requests (those from pi -> laptop) will not get any response. This is because by default Windows Firewall blocks these requests from being answered. To disable this, run a PowerShell prompt as Administrator and enter these commands:
    ```
    New-NetFirewallRule -DisplayName "Allow inbound ICMPv4" -Direction Inbound -Protocol ICMPv4 -IcmpType 8 -Action Allow
    
    ```

    New-NetFirewallRule -DisplayName "Allow inbound ICMPv6" -Direction Inbound -Protocol ICMPv6 -IcmpType 8 -Action Allow
    ```

## Troubleshooting

Troubleshooting for the ISC DHCP Server can be found under [troubleshooting dhcp](/resources/manage-dhcp/).