# DHCP
> DHCP advice will be updated and rolled out to this page.
> Reminder that the service **`dhcpd`** will handle dhcp for us.

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

###  General Troubleshoot
---

> Follow the checklist below if you had dhcp previously working and it recently failed.

- Check DHCP errors `journalctl -u isc-dhcp-server`
- check subnet range in `dhcpd.conf` 
-  Check what wifi your pi is using the appropriate wifi
   -  `wpa_cli -i wlan0 status` 
-  Make sure both `dhcpd` and `systemd` are declaring the same statis IP for the Pi.
-  Check your leases in `at /var/lib/dhcp/dhcpd.leases`


### Troubleshooting ISC DHCP Server
---

> Follow steps below if you are having trouble getting your ISC DHCP server setup for the first time.

Raspbian provides us with several commands to troubleshoot and find errors related to services. Check status and recent log output using `systemctl status isc-dhcp-server.service` or `journalctl -xe`  .

Search the system logs for relevant errors `journalctl -u isc-dhcp-server`

As you debug, keep the following points in mind:

* A misplaced space or bracket may cause DHCP to fail, so pay close attention to syntax.
* Your Pi will keep it's static address, so be sure that you excluded the address from the DHCP pool.
* Double check that you’ve configured the server defaults with the correct interface names and commented out IPv6 related settings.
