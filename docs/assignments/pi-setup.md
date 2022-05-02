# Raspberry Pi Setup Guide (2022-03-29)
## Overview 

This guide will walk you through the steps necessary to install Raspberry Pi OS on your Pi and connect it to Wifi. While we encourage you to search online and use other resources when you encounter questions, it's important that you follow our instructions closely. Though there are many different guides that can help you accomplish the same objectives, this guide has been built based on the needs and requirements of our projects and introduces you to tools you will need throughout the quarter.

The core sections of this guide will walk you through the following steps:

1. Install the Raspberry Pi OS on a microSD
1. Enable SSH for _headless_ management
1. Set locale and other base configuration
1. Configure wifi connections
1. Update Raspberry Pi OS and installing additional packages with `apt`

To complete these steps, you will need all of the following items:

1. Computer running a recent desktop OS, such as: macOS 11+, Windows 10+, or a desktop Linux distribution
1. Ethernet network interface (built-in or USB adapter)
1. Media card reader supporting microSD (built-in or USB adapter)
1. Unused 8GB+ microSD card (you may repurpose an existing card, but be aware that all contents will be wiped)

Before you start setting up your Pi, please review the tasks in the following section. These steps are particularly important for students running Windows and Linux.

## Before you start

### Windows Users

#### Enable the built-in OpenSSH Client
All current versions of Windows 10 and 11 ship with an OpenSSH client, though the feature may be disabled by default. Follow the instructions here to enable this feature.

1. Navigate to _Settings_ -> _Apps_ -> _Optional features_
1. Click on _Add an optional feature_
1. Enter _OpenSSH_ and select the _OpenSSH Client_
1. Click next to install

#### Text Editor
Among the many subtle differences between Windows and Unix-based machines is a difference in the control characters used to terminate lines in text files. In many cases, this difference will prevent Linux from parsing a file you've written and copied from Windows.

To avoid this issue, install a code-oriented text editor that can be configured to use Unix-style line endings and use it exclusively for the labs and projects throughout this course. __Visual Studio Code__ and __Atom__ are common options.

To configure __VS Code__ to use Unix-style line endings:

1. Open Settings
1. Search for the term “Eol”
1. Change the default end of line character to _\n_

To configure __Atom__ to use Unix-style line endings:

1. Open Settings
1. Navigate to Packages / Line Ending Selector
1. Change the _Default line ending_ to _LF_

#### Terminal
Except when otherwise noted, I recommend that Windows users complete all Linux networking exercises within Windows PowerShell rather than Git Bash or the Command Prompt.

!!! info Recommendation
    For a much improved working experience, you should use the new __Windows Terminal__ application. Terminal is available in the __Microsoft Store__ for Windows 10 users and it is shipped pre-installed with Windows 11.
    
    By default, the terminal will run PowerShell, but it also supports Command Prompt and Windows Subsystem for Linux. 

### Linux Users
Using your default package manager (likely `yum` or `apt`), confirm that __Avahi mDNS__ services are installed (mDNS is typically part of the default distribution).

## Install Raspberry Pi OS
The Raspberry Pi is a single-board computer built with Linux distributions in mind. The official operating system, which we'll use in our labs is known as __Raspberry Pi OS__ and is based on Debian Linux. If you're familiar with Ubuntu or Arch Linux, you should be mostly at home working in Raspberry Pi OS. Don't worry if Linux is not your jam. We'll provide plenty of guidance so that you can focus your energy on network concepts.

### Write Raspberry Pi OS to MicroSD

1. Download the Raspberry Pi Imager from https://www.raspberrypi.com/software/ and launch the software
1. Click the button labeled _CHOOSE OS_
1. Select __Raspberry Pi OS (other)__ and scroll down to find __Raspberry Pi OS Lite (64-bit)__
1. Insert a microSD card into your card reader and proceed with the install.

Unlike the full version of the operating system, __Raspberry Pi OS Lite__ is completely text-based and is well-suited for _headless_ operation. There isn't a graphical desktop environment, but you won't need it. All of projects in this course will be completed in a remote command-line interface (CLI).

### Update Configuration
Etcher will eject the microSD once the image is completely rewritten. We want to edit some files on the SD, so you will need to briefly remove the card before inserting it again.

On macOS or Windows, you'll be limited to accessing the boot partition of the card. Use Explorer or Finder to locate and open the partition. 

You must complete the following step before the first boot.

#### Enable SSH
Due to security considerations, the newest versions of Raspian disable SSH by default, but it's easy to turn the feature on so that we can connect to the Pi without requiring a monitor and keyboard.

To enable SSH on the first boot, add an empty file named ssh to the boot volume. There are many ways to create this file. If you want to use the command-line, follow the instructions below (macOS and Windows):


=== "macOS"
    ``` shell-session
    # Unix-based systems mount external storage to a path in the directory tree. For a freshly written Raspberry Pi OS image, this path will be /Volumes/boot.

    touch /Volumes/boot/ssh
    ```
=== "Windows"
    ```pwsh-session
    # Windows mounts external storage to a drive letter. Replace E: with the letter assigned on your system.

    New-Item -type File E:\ssh
    ```

Raspberry Pi OS checks for /boot/ssh during startup. When present, the OS enables the OpenSSH _daemon_ and deletes the file. 

!!! information
    _Daemon_ is a computing term used to describe software that runs in the background, e.g., to provide a service for network-based clients. The term is most commonly associated with Unix-flavored operating systems.

## Initial Boot
It's time to boot the Pi for the first time. Close your editor and any windows that are open to the microSD so that you can eject the card gracefully from your OS. 

1. Remove the microSD and insert into the card slot on your Pi.
1. Connect power to the designated micro-USB port.
1. Attach to your computer with an Ethernet cable and get ready to launch an SSH connection.
1. From terminal or PowerShell, connect to the Pi for the first time by running the following command:

```bash
# The default password for pi is raspberry
ssh pi@raspberrypi.local
```

This command directs your local SSH client to connect to a network host named `raspberrypi.local` with the username `pi`. Since you haven't connected to this device before, SSH should ask you to accept the connection of an unknown device before presenting you with a password prompt. 

!!! information
    Before the SSH client can initiate it's connection to the Pi, it must find the IP address associated with _raspberrypi.local_. This process is called resolution and will be completed using a protocol known as multicast DNS (mDNS). 

``` bash-session
$ ssh pi@raspberrypi.local
The authenticity of host 'raspberrypi.local (fe80::1b9c:bcf2:acd6:bbbe%42)' can't be established.
ECDSA key fingerprint is SHA256:QqhpMybvctuIxV03xcnlANU3cxWM1JhvSYxloSd69Rw.
Are you sure you want to continue connecting (yes/no)?
```

If you had previously connected to a different host with the name `raspberrypi.local`, you may also see the warning shown below (please refer to the troubleshooting section to resolve this error):

<div class="local">
``` text
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```
</div>

After confirming the prompt shown above, you will see another short message followed by a password prompt.

```
Warning: Permanently added 'raspberrypi.local,fe80::1b9c:bcf2:acd6:bbbe%42' (ECDSA) to the list of known hosts.
pi@raspberrypi.local's password:
```

The default password for the `pi` user is `raspberry`. Once login is complete, you should be greeted with a message similar to that shown here.


``` bash-session
Linux raspberrypi 4.14.79-v7+ #1159 SMP Sun Nov 4 17:50:20 GMT 2018 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Linux raspberrypi 4.14.79-v7+ #1159 SMP Sun Nov 4 17:50:20 GMT 2018 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

Wi-Fi is disabled because the country is not set.
Use raspi-config to set the country before use.

pi@raspberrypi:~ $
```

### Choose a New Password
Before proceeding further with setup, you should change the default password by entering the `passwd` at the command prompt. You will be guided through three prompts to enter the current password and then to update/confirm the new password.

!!! danger
    SSH is a favorite target of malicious attackers. Nothing makes their lives easier than devices that are configured with the default password for the type of system.

### Default Configuration

Several additional steps are required to prepare your Pi for future projects. While some of these steps may not seem essential, omitting them might pose consequences when trying to troubleshoot problems later in the course. 

Setting the timezone will ensure that log messages are displayed in local time, which is quite helpful for troubleshooting.

``` bash-session
# You can see a list of timezones by running `timedatectl list-timezones`
sudo timedatectl set-timezone America/Los_Angeles
```

Raspberry Pi OS defaults to a British locale. This can cause issues if we ever need to troubleshoot your device using an external keyboard. Configure the locale and keymap to prevent these issues.

``` bash-session
# Update /etc/locale.gen with your preferences
sudo nano /etc/locale.gen # Find and uncomment the en_US.UTF-8 locale 

# Re-generate locale information after updating the locale.gen file
sudo locale-gen 

# Apply new settings
sudo localectl set-locale "LANG=en_US.UTF-8"
sudo localectl set-keymap us
```

Occasionally we'll end up running a program that uses the default editor settings for your profile (or root if we have run the program with _sudo_). It's helpful to have these configured so that you don't end up stuck in _vi_ or _ed_ without any way to exit.

``` bash-session
# Update default editor selections for the pi user
select-editor

# Modify the same setting for commands that require root privileges.
sudo select-editor
```

### Retain Logs Between Boots
In recent versions of Raspberry Pi OS, system logs are collected by the **journald** service. In the default configuration, logs are not retained between reboots of your Pi.

To ensure that logs are available when you need to debug, you can enable log retention by running the following commands.

``` bash-session
# Create the location for persistent log messages
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
```

### Update Hostname
At first boot, we could locate our Pi on the network based on the default hostname of **raspberrypi.local**. This is fine in an isolated, point-to-point network, but it's a problem when we connect to shared networks. 

Use the `hostnamectl` command to set a _unique_ name for your device.

``` bash-session
# This little pi likes to be called Titan
sudo hostnamectl set-hostname titan
```

The main change that **hostnamectl** makes is visible in `/etc/hostname`. We also need to update references to the hostname in `/etc/hosts`. The **hosts file** is present on most Operating Systems and provides a way of defining hostname/IP address associations without relying on DNS. Linux uses this file to associate your hostname to the loopback IP address.

``` bash-session
# Confirm your hostname is up to date in /etc/hostname
cat /etc/hostname

# Replace any references to raspberrypi in /etc/hosts
sudo nano /etc/hosts
```

??? faq "FAQ: Why is `sudo` slow"
    You might notice odd warnings and sudo latency immediately after changing the hostname. These should clear up once you update the **hosts file**. For reasons beyond the scope of this curriculum, **sudo** performs DNS lookups as part of its check to determine whether the current user is permitted to elevate privileges. The delay you experience is the tool waiting for a DNS response prior to its timeout.

In order for the _hostname_ changes to take full effect, reboot your pi by calling `sudo reboot now`. After 20 - 30 seconds your pi will be visible with the new name -- you will need to include the `.local` suffix on any command referencing the name.

??? example
    ``` bash-session
    # ping by mDNS hostname
    ping titan.local

    # ssh by mDNS hostname
    ssh pi@titan.local
    ```

## Enable Passwordless Login
We can significantly simplify the process of managing our Pi’s by copying our public SSH keys into the `authorized_keys` file of the default pi user.

To prepare the Raspberry Pi, we need to create a remote directory under the `pi` user's home directory. If you're still logged into your pi, you can create the directory using `mkdir ~/.ssh`. After that is done, type `exit` to terminate the connection and return to your local command prompt.  

??? faq "FAQ: Why aren't `ssh` and `scp` working"
    Please review the previous set of instructions closely. The commands below are run from your **local** shell. Running them inside of an existing SSH session will not have the effect you are looking for.

With the directory in place, copy your public key (most likely `~/.ssh/id_ed25519.pub`) to the pi using the `scp` command.

``` bash-session
scp $HOME/.ssh/id_ed25519.pub pi@raspberrypi.local:.ssh/authorized_keys
```

!!! info
    The second set of parameters in this command specifies the user (`pi`) the host (`titan.local`) and the location of the new file (`.ssh/authorized_keys`). Like `ssh`, `scp` is a user-based tool and will be executed relative to the home directory of the authenticated user (`/home/pi`). We can override the default by specifying an absolute path beginning with a `/`, e.g., `/home/pi/.ssh/authorized_keys`.

<br>
<br>

## Connect to Wifi
---
To complete this guide, you will need to establish Internet connectivity for your Pi. Since the Pi 4 has integrated wireless capabilities, we can solve the problem by connecting to local wifi.

### Set the Wireless Network Locale
Wireless networks operate in radio frequency (RF) ranges that are subject to regulation in the geographic region that the network is operated. Before turning on the wireless features of your Raspberry Pi, set the regulatory domain to the United States.

There are several ways to modify wireless settings on your Pi. For now, we'll use the `wpa_cli` command to interactively modify the settings. 

``` bash-session
# Launch wpa_cli interactively
wpa_cli -i wlan0

# Enter the following commands at the wpa_cli prompt

get country 
# Returns two letter code for current setting regulatory domain

set country US
# Returns OK

save_config
# Returns OK

quit
```

### Enable the Wireless Interface
Recent versions of the operating system use a system feature named __[rfkill](https://01.org/linuxgraphics/gfx-docs/drm/driver-api/rfkill.html#rfkill-rf-kill-switch-support)__ to disable the WiFi radio by default. Before you can connect to any networks, you'll need to remove this block.

``` bash-session
sudo rfkill unblock wlan
```

### Configure the WPA Supplicant
Wireless settings for the Pi are controlled by a service called _wpa_supplicant_, which stores network connection settings inside `/etc/wpa_supplicant/wpa_supplicant.conf`. You already saw an example of modifying these settings through `wpa_cli`. For the following task, I recommend that you edit the file directly in a text editor of your choice.

Before you make any changes, take a look at the default configuration by running `sudo cat /etc/wpa_supplicant/wpa_supplicant.conf` (sudo is required since this file is only readable to root). You should see something like this:
  
``` text
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
```

If you are editing the file on your own computer, paste these lines at the top of the file and add the rest of your changes below them. You will be adding a network definition for each WLAN that you want to use. Detailed configuration instructions are available in [WPA Supplicant Configuration Reference](/resources/wifi-reference/){target=_blank}.

For this project, use the configuration reference to set up the following networks:

1. [Eduroam (w/ Certificate Based Authentication)](/resources/wifi-reference/#wpa2-enterprise-networks){target=_blank}
1. [(Optional) UW MPSK Network](/resources/wifi-reference/#wpa2-personal-networks){target=_blank}
    - [Requires Device Registration](/resources/uw-wireless/#uw-mpsk-network-onboarding){target=_blank}
1. [(Optional) Home Network](/resources/wifi-reference/#wpa2-personal-networks){target=_blank}

### Testing Wireless Connections
When you are finished updating `wpa_supplicant.conf` reconfigure the wireless interface by calling `wpa_cli -i wlan0 reconfigure`. The command should return `OK` after a few seconds. Check that you are attached to the wireless network by calling `wpa_cli -i wlan0 status`.

!!! information
    You previously ran the _wpa_cli_ tool in interactive mode. The instructions above pass individual commands and return you directly to the Linux command prompt. This method is quick and convenient, but if you run into problems, interactive mode will provide the most information for debugging purposes.

??? example "Example of a successful connection"
    ``` text
    wpa_cli -i wlan0 status

    <3>Trying to associate with SSID 'eduroam'
    <3>Associated with 1c:28:af:07:55:31
    <3>CTRL-EVENT-EAP-STARTED EAP authentication started
    <3>CTRL-EVENT-EAP-STATUS status='started' parameter=''
    <3>CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
    <3>CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=13
    <3>CTRL-EVENT-EAP-STATUS status='accept proposed method' parameter='TLS'
    <3>CTRL-EVENT-EAP-METHOD EAP vendor 0 method 13 (TLS) selected
    <3>CTRL-EVENT-EAP-PEER-CERT depth=2 subject='/C=US/ST=New Jersey/L=Jersey City/O=The USERTRUST Network/CN=USERTrust RSA Certification Authority' hash=e793c9b02fd8aa13e21c31228accb08119643b749c898964b1746d46c3d4cbd2
    <3>CTRL-EVENT-EAP-PEER-CERT depth=2 subject='/C=US/ST=New Jersey/L=Jersey City/O=The USERTRUST Network/CN=USERTrust RSA Certification Authority' cert=308205de308203c6a003020102021001fd6d30fca3ca51a81bbc640e35032d300d06092a864886f70d01010c0500308188310b3009060355040613025553311330110603550408130a4e6577204a6572736579311430120603550407130b4a65727365792043697479311e301c060355040a131554686520555345525452555354204e6574776f726b312e302c06035504031325555345525472757374205253412043657274696669636174696f6e20417574686f72697479301e170d3130303230313030303030305a170d3338303131383233353935395a308188310b3009060355040613025553311330110603550408130a4e6577204a657273657931...
    <3>CTRL-EVENT-EAP-PEER-ALT depth=0 DNS:radius.uw.edu
    <3>CTRL-EVENT-EAP-PEER-ALT depth=0 DNS:radius.washington.edu
    <3>CTRL-EVENT-EAP-STATUS status='remote certificate verification' parameter='success'
    <3>CTRL-EVENT-EAP-STATUS status='completion' parameter='success'
    <3>CTRL-EVENT-EAP-SUCCESS EAP authentication completed successfully
    <3>CTRL-EVENT-CONNECTED - Connection to 1c:28:af:07:55:31 completed [id=0 id_str=]
    ```

### Verify Connection
In addition to `wpa_cli`, there are several tools that can be used to check the current state of your wireless interface and determine whether it has received a valid configuration from DHCP.

``` bash-session
# Show the current state of the wlan0 interface
ip link show wlan0

3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DORMANT group default qlen 1000
    link/ether b8:27:eb:db:fe:93 brd ff:ff:ff:ff:ff:ff

# Show address configuration for the wlan0 interface
ip addr show wlan0

3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether b8:27:eb:db:fe:93 brd ff:ff:ff:ff:ff:ff
    inet 10.18.185.176/15 brd 10.19.255.255 scope global wlan0
       valid_lft forever preferred_lft forever
    inet6 fe80::ce14:df50:b3f0:9c2a/64 scope link
       valid_lft forever preferred_lft forever
```

## Update Software Packages
---
Let's finalize the initial setup by checking for updates to Raspberry Pi OS and its default packages (this can take a few minutes on slow networks).

As a Debian based Linux distribution, Raspberry Pi OS relies on `apt` for package management. The `apt update` command is used to determine whether there are new packages to download.

``` bash-session
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade

# We've found that an additional update is sometimes needed after a dist-upgrade
sudo apt update

# Install some useful packages while you're at it
sudo apt install dnsutils
```

## Graceful Shutdown
Never yank the power from your Pi when it's time to quit. This can lead to data corruption on the microSD (they aren't as resilient as the drives installed in your laptop). 

Instead, you should always issue a proper shutdown via SSH.

From an SSH connection, run `sudo poweroff`. If you ever need to reboot the Pi, call `sudo reboot now`.

Wait about 15 - 20 seconds to allow the Pi to complete it’s shutdown process before disconnecting the power.


<br>
<br>


## Tips and Troubleshooting
---

### Can't find raspberrypi.local
If you can't make an initial connection to the Raspberry Pi, verify the following requirements were satisfied:

- Raspberry Pi OS has been successfully written to microSD card
    - Test again with a known good SD card
- [OpenSSH daemon was enabled](#enable-ssh) before booting the Pi[^ssh_gone]
- Physical connection is healthy
    - Make sure that both ends of the cable are completely inserted
    - Try an alternate cable, or test the cable with another device
- Network interface is functional
    - Check for link lights on both network interfaces
    - Verify that your network adapter is visible to the OS
    - Determine whether your network adapter requires [drivers](#missing-network-drivers)
- Network interface is assigned a [link local address](#link-local-addresses)
- Current system configuration supports [multicast DNS (mDNS)](#multicast-dns)

[^ssh_gone] Raspberry Pi OS will remove the /boot/ssh file after enabling the service. You do not need to restore the file if you find that it has disappeared.

#### Missing Network Drivers
If you are using a USB Ethernet port for the first time on your computer, it's possible that your first attempt to connect will be unsuccessful. Some issues are resolved simply by disconnecting the USB briefly and trying to connect again after you have plugged everything back in. 

If your issues persist, check the manufacturer's support page or search to determine whether your computer needs additional drivers for the specific model of Ethernet adapter. If drivers are necessary, they will usually be available from the manufacturer's website.

#### Link Local Addresses
Run `ipconfig` on Windows or `ifconfig` on other devices to confirm that the interface registers a network connection. The local IPv4 address for this interface should begin with `169.254.x.x` (link-local address range). Most devices will assign this type of address automatically if a DHCP cannot be found.

**Linux** Some versions of Linux, e.g., Ubuntu, require you to choose between link-local addressing and DHCP. Explore your advanced adapter settings for Ethernet to change the adapter settings.

#### Multicast DNS

**Windows** Make sure that you are on a recent build of Windows 10 or Windows 11. Though mDNS is now a draft standard, Microsoft did not include a full client until late 2018. Support for mDNS on earlier versions of Windows relied on the Bonjour Print Services software that was once distributed by Apple in conjunction with iTunes and various other applications. 

**Linux** Desktop Linux distributions often support mDNS resolution using the Avahi daemon. Review the requirements at the [beginning of this guide](#verify-support-for-mdns).

### SSH Connection Warning 
At some point, you are likely to receive the following warning when trying to launch `ssh`. In most cases, what you are seeing is expected behavior. 

``` text
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

The first time you connect to a host, SSH will bind a cryptographic signature to the hostname, e.g., `raspberrypi.local` and store it within `~/.ssh/known_hosts`. To prevent attacks, SSH warns us when the signature changes for a known host. 

In our case, it's likely that there is another explanation for the mismatched signature since SSH will create a new private key every time we set up or rebuild a Raspberry Pi.

Since this is not an attack scenario, resolve the conflict with `ssh-keygen -R raspberrypi.local`, which will remove the existing entry. 

!!! bug "Missing features in Windows SSH"
    The `ssh-keygen -R` feature isn't present before Windows 10 1809. If you are still running an older version of Windows, you will have to locate `known_hosts` within the `.ssh` configuration directory and remove the line indicated by the warning message.

### Wireless Network is Disabled

Check the state of your wireless network interface using the `rfkill list`. If a soft-block is listed for `wlan0`, it's likely that you skipped one or more previous steps. Double-check your [wireless locale](#set-the-wireless-network-locale) and use rfkill to [enable your interface](#enable-the-wireless-interface). 

### Permission Denied Errors

When we're performing system and network administration, we are often in need of **root** privileges. Running an administrative command without the appropriate privileges will result in a permissions-related error.

By default, the **pi** user is restricted in which parts of the system it can read and modify (Linux is heavily user-based in it's permissions model); however, the user is permitted to elevate itself to the admin level with `sudo` when it is necessary to perform privileged commands.

As a general security precaution, don't `sudo` commands unless it's necessary. For now, we'll indicate when that's the case. In later tasks, you will need to make this determination on your own.
