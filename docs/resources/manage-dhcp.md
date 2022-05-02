# DHCP
> DHCP advice will be updated and rolled out to this page.

###  Renew dhcp lease
---

!!! quote 
    > dhcp renewal is when a dhcp client (your laptop, phone, tablet...) renews or updates its IP address cofniguration with the dhcp server. Often times in our homes the dhcp server is already packaged in our router [source](https://www.serverbrain.org/network-services-2003/how-the-dhcp-lease-renewal-process-works-1.html).

**Unix (Mac OS)**: <br>
> This steps below are necessary since the renew button for mac os only does a shortened DHCP exchange.

* Go to Network Preferences >> Select interface >> Advanced -> TCP/IP, Turn off IPv4 Addressing (hit OK + Apply)
* Return to advanced settings and enable DHCP. 

**Windows**:

* Open a command prompt
* run `ipconfig /release Wi-Fi` to release your IP address
* then run `ipconfig /renew` to request a new one

**Linux**: 

* `sudo dhclient -r eth0`
* `sudo dhclient eth0`



