# Networkd Setup for Raspberry Pi OS (2020-01-22)
## Intro to Systemd
An **init provider** is the first process that is launched on a Linux system. It is responsible for loading other essential services, and it ultimately becomes the parent of every other process on the system. Among the differences between different Linux distributions is the choice of init system.

**systemd** is the initialization (init) provider most often deployed on recent Linux distributions. On Raspberry Pi OS and other recent Debian-based distros  **systemd** replaces the aging **sysvinit**. We will interact directly with **systemd** in this tutorial by issuing **`systemctl`** commands to start, stop, enable and disable various system-level services.

Beyond its job of launching essential services, **systemd** provides a modular interface to control many parts of the system. We've already interacted with a few of these components when we used **`localectl`**, **`timedatectl`**, and **`hostnamectl`** during the initial setup of the RaspberryPi.

In this tutorial we will leverage three more **systemd** subsystems:

- __`journald`__ - Collects and catalogs log files from system services and applications. You can explore these files with **`journalctl`** as seen in [Troubleshooting](#troubleshooting).
- __`networkd`__ - Detects and configures network devices and performs various other network management functions.
- __`resolved`__ - Provides network name resolution and maintains a list of DNS servers (network-based resolvers) to contact based on settings provided by DHCP or **`networkd`** configuration files.

!!! danger "Attention: Risk of _bricking_ your Pi"
    A typo or other mistake within the core network configuration can result in total loss of network connectivity, which is required to manage the Pi over ssh.

    Before you begin to follow these instructions, make an external copy of critical configs such as **wpa_supplicant** using **`scp`** in case it becomes necessary to rebuild. 
    
    If you get locked out during this process, please contact the instructor to determine whether it's possible to reconnect without re-imaging the Pi.


## Systemd vs Default Networking
By default, Raspberry Pi OS networking relies on a component called dhcpcd to manage interface settings, addresses, etc. While this component works well for general purpose computing, but it is not a match for every scenario.

The current version of Debian (the base OS for Raspberry Pi OS) supports **systemd** based network management via the **systemd-networkd** service. Unlike **dhcpcd**, **networkd** is well-suited to managing the base configuration for our router.

## Enabling Networkd
We will enable **networkd** in several steps. Try to complete this process in one sitting to avoid "bricking" your Pi (at least making it inaccessible over the network).

### Configure network interfaces
To configure the network with **networkd**, we create files in **`/etc/systemd/network/`** that match named network interfaces and define our desired settings settings. Create the following **`.network`** files to handle the default settings for wired and wireless interfaces. Enabling link local addresses is a critical component of these default configurations so that you can still communicate with the Pi when DHCP is not available.

!!! warning
    **Networkd** configuration is generally easy to work with, but there are a couple of footguns to be aware of:

    - The files are case sensitive and sometimes sensitive to seemingly innocuous changes in spacing. 
    - A file with a missing or invalid [Match] statement will be applied to every interface that hasn't already been matched by an earlier configuration.

!!! faq "FAQ: **Networkd** configuration files"
    **`man systemd.network`** provides a comprehensive reference to the format of the **networkd** configuration files and the process for loading them. 
    
    At runtime, **networkd** gathers files using the **`.network`** extension from several locations, including **`/etc/systemd/network/`**. To determine the order in which to apply configuration, **networkd** sorts all of these files lexically. 
    
    To maintain proper control over execution order, we can prepend a two digit numeric value to the filename. This allows us to set up high-numbered default configurations that are overriden on a case-by-case basis by low-numberd configs files.

#### **Default ethernet configuration**
__`/etc/systemd/network/99-eth.network`__
```
[Match]
# Applies to eth0, eth1, ...
Name=eth*

[Network]
# Configure IPv4 using DHCP
DHCP=ipv4

# Enable link local addresses for IPv4 (169.254.x.y) and IPv6 (FE80::)
LinkLocalAddressing=yes
```

#### **Default wireless configuration**
__`/etc/systemd/network/99-wlan.network`__
```
[Match]
Name=wlan*
[Network]
DHCP=ipv4
```

### Disable default networking

!!! tip "Tip: Backup configuration before proceeding"
        If you've made any mistakes, you may lose access to your Pi after performing the following steps. 
        
        Make an offline backup **before** you proceed by copying files to your local system with **`scp`**.

```
# Disable default Raspberry Pi OS networking
sudo systemctl mask dhcpcd.service

# Disable legacy Debian networking
sudo systemctl mask networking.service

# Disable resolvconf service, which manages DNS servers for the OS
sudo nano /etc/resolvconf.conf
# Add the line resolvconf=NO to the beginning of the file
```

### Enable systemd network services
```
# Enable systemd-networkd to replace Raspberry Pi OS and Debian networking
sudo systemctl enable systemd-networkd

# Enable systemd-resolved to replace the resolvconf service
sudo systemctl enable systemd-resolved
```

DNS resolvers for Linux are loaded from the **`/etc/resolv.conf`**. Rather than manage this directly or through **resolvconf**, we want to load resolvers dynamically from **systemd-resolved**. This is as simple as creating a link 

```
# Set up a symbolic link from systemd-resolved resolv.conf to the system resolv.conf
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

### Update Wireless Service
In order for **wpa_supplicant** to work correctly with **networkd**, one further change is needed. In our original configuration, we created a system-wide configuration file for wireless. This setup works just fine with the default networking setup, especially since the Pi is only equipped with one wireless interface by default. For **networkd** to work properly with **wpa_supplicant**, we need to attach our configuration directly to the **wlan0** interface as shown below.

```
# Rename configuration as a per-interface file
sudo mv /etc/wpa_supplicant/wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant-wlan0.conf

# Enable the wpa_supplicant service specifically for wlan0
sudo systemctl enable wpa_supplicant@wlan0

# Disable the system-wide service
sudo systemctl disable wpa_supplicant
```

## Test Your Configuration
Call **`sudo reboot now`** to reboot the Pi. Log back in and verify that your Pi is fully operational:

- Confirm that your wireless interface is online
- **`cat /etc/resolv.conf`** to confirm that it is being populated by **systemd-resolved**

## Troubleshooting
### General Guidance
When logging back into your Pi, you may find that some of the functionality is failing. In addition to validating log files and checking that you completed all instructions, log files can provide valuable information for detecting where an error is occurring.

We have already mentioned **journald** earlier in this guide. In order to evaluate logs that are created by **journald**, we make use of the **`journalctl`** command. [Read up](https://www.lifewire.com/what-to-know-less-command-4051972) on the **`less`** utility to learn how to efficiently navigate **`journalctl`** output.

```
# Show log messages
journalctl

# Include messages marked secure
sudo journalctl

# Show messages from last boot
journalctl -k -b -1

# Show networkd messages
journalctl -u systemd-networkd

# Show resolved messages
journalctl -u systemd-resolved

# Show wpa_supplicant messages
journalctl -u wpa_supplicant

# Learn more about journald
man journalctl
```

### Accessing a Bricked Pi
If you made a mistake at the wrong point in the previous process, you may not even be able to connect anymore from the network. 

If this happens, there are several ways to restore access:
- Contact instructor for a repair script. Raspberry Pi OS provides a mechanism to run a script during the boot process. You will need access to a microSD reader/writer in order to add the script into the boot folder.
- Connect a USB keyboard and HDMI monitor to the Pi in order to debug directly. You will need a micro-HDMI adapter. Most flat-screen TV's can be used as a monitor for this purpose.
- Use a serial console cable to access the Pi command line directly. Come to office hours or contact instructor for this option.
- Mount the SD card using a Linux virtual machine and repair the broken config files directly.

### Rebuilding Raspberry Pi OS
Alternatively, you may opt to reflash your memory card and repeat initial setup. Contact the instructor before proceeding. 

If you find yourself in this position, the following recommendations will simplify the process:

1. Copy a fully configured **`wpa_supplicant.conf`** to the **`boot`** volume of the memory card before the first boot. The boot process will move the file to the proper location in **`/etc/wpa_supplicant`** and set its permissions on the first boot.
2. Remove the previous server key hash from **`.ssh/known_hosts`** to prevent **`ssh`** from complaining about the Pi's new key.
    - __macOS/Linux__ - Run **`ssh-keygen -R raspberrypi.local`**
    - __Windows__ - Edit **`$HOME\.ssh\known_hosts`** and remove the lines that reference **`raspberrypi.local`** before you attempt to connect.
3. __macOS/Linux__ - Run **`ssh-copy-id -i ~/.ssh/id_ed25519`** to quickly copy your public keys into **`authorized_keys`**. _Windows_ users will have to manually set this file up as shown in previous guides.
