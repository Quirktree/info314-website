# Mac/Windows/Linux Networking



## Renewing DHCP Leases

> dhcp renewal is when a dhcp client (your laptop, phone, tablet...) renews or updates its IP address configuration with the dhcp server. Often times in our homes the dhcp server is already packaged in our router [source](https://www.serverbrain.org/network-services-2003/how-the-dhcp-lease-renewal-process-works-1.html).

### macOS

!!! important
    The _Renew DHCP Lease_ button for macOS will not result in a complete DHCP Exchange. Be sure to follow the instructions below.

* Go to Network Preferences >> Select interface >> Advanced -> TCP/IP, Turn off IPv4 Addressing (hit OK + Apply)
* Return to advanced settings and enable DHCP (hit OK + Apply). 

### Windows

* Open a command prompt
* Run `ipconfig /release Wi-Fi` to release your IP address
* Wait for the release command to complete and run `ipconfig /renew` to request a new one

### Linux

!!! Important
    Linux instructions will vary depending on your target distribution. You may need to search online for alternative instructions.

* `sudo dhclient -r eth0`
* `sudo dhclient eth0`



## Managing ARP Cache

Every computer maintains a cache of associations of ARP responses that have recently appeared on the network. In order to examine ARP functionality or troubleshoot the behavior of a particular network device, you may need to flush the existing contents.

### Windows

Open an Administrator Command Prompt or PowerShell Session and run`arp -d`

### Unix (MacOS)

`sudo arp -ad` 

### Linux 

For the following command, substitute the name of your own network interface for **<wlan0>**: 

`sudo ip neigh flush dev <wlan0>`