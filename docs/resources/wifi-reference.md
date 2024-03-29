# WPA Supplicant Configuration Reference (2022-03-29)

## Basic Configuration

Wireless settings for the Pi are controlled by a service called **wpa_supplicant**, which stores information about known wireless networks in a text-based configuration file in **`/etc/wpa_supplicant/`**. By default, in Raspberry Pi OS, the name of this file is **`wpa_supplicant.conf`**. You may also encounter an interface-specific configuration where the interface name is appended to the file, e.g., **`wpa_supplicant-wlan0.conf`**. In both cases, the file is owned by **root** and requires **root** privileges to read or edit. To learn more about **wpa_supplicant**, run the command ` 

??? danger "Security Risk"
    **`wpa_supplicant.conf`** is protected from casual reading due to the fact that it is a sensitive file that might contain the password to your home network or even the password for your school/work account.

    Practice caution with this file:
    
    1. Never share the contents of this file directly online.
    1. Scrub passwords, keys, and hashes before committing a copy into a **repository**.
    1. Reset passwords if you lose your Pi or suspect that you may have disclosed it inadvertently.

You can edit **wpa_supplicant** configs directly on the Pi using any terminal-based text editor. Alternatively, you can create the file on your local system and copy it into place on the Pi (as described later in this guide). 

??? warning "Warning: Windows line-endings and rich text format"
    As students get started with Linux networking, we frequently encounter problems related to the overall file format. As a rule, the configuration files you create in this class must be plain text with standard line endings. 
    
    You'll want to stick to using a code-oriented text editor, as opposed to options like macOS TextEdit and Windows Notepad that often save files as _rich text_ rather than plain text.
    
    For Windows users, a code-oriented editor will also help you avoid issues related to line endings. While most operating systems use a simple _New Line (aka Line Feed)_ control character to signify the end of a line, most Windows tools also include a _Carriage Return_. This alternate line ending causes parsing errors in many Linux tools.

The general structure of the configuration file is show below. The file begins with a standard set of parameters specifying the **country** (needed to initialize appropriate radio settings), a **control interface** used by network management tools, and a boolean that instructs **wpa_supplicant** to accept configuration updates from other network management tools. We won't delve any deeper into the meaning of these initial parameters for now. Rather, our concern will be how to configure Linux to join nearby wireless networks.

!!! example "Example `wpa_supplicant.conf`"
    ```
    country=US
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1

    network={
        ssid="Some public network"
        key_mgmt=NONE
    }
    
    network={
        ssid="My home network"
        psk="Don't tell anyone the password"
    }
    ```

Each network is represented by a configuration block that surrounded by `network={...}`. The contents of the network block will depend largely on the security settings of the network, e.g., whether or not the network is encrypted with a passphrase. This document provides instructions for configuring three common types of networks: 

1. Unencrypted Networks
2. WPA2 Personal Networks (simple passphrase)
3. WPA2 Enterprise Networks

Let's start by examining the configuration for an unencrypted network.

##  Unencrypted Networks

All networks are defined by `parameter=value` pairs enclosed within `network={}`. Regardless of security configuration, each network block is required to contain an **`ssid`** parameter identifying the network. The **service set identifier (SSID)** is the network name that you see on your device when you connect to a wireless network. Since this name may include whitespace, we encapsulate it in double quotes.

In addition to the **ssid**, **wpa_supplicant** expects us to provide encryption a passphrase and other encryption settings for the wireless network. Omitting these settings, even for an unencrypted network, will result in errors. 

Instead, for unencrypted networks, we explicitly disable encryption with **`key_mgmt=NONE`**.

!!! example "Configuration for an unencrypted network"
    ```
    network={
        ssid="Coffee Shop"
        key_mgmt=NONE
    }
    ```

## WPA2 Personal Networks
For a basic (non-enterprise) encrypted network, the configuration of the network block changes only slightly. Rather than specify the `key_mgmt` setting, we assign the network passphrase to the `psk` parameter.

There are two ways to accomplish this task. First, we can assign the passphrase directly to the parameter in plaintext as shown here:

!!! danger "Danger: Don't do this!!!"
    ```
    network={
        ssid="Home Wifi"
        psk="super secret squirrels"
    }
    ```

Security professionals generally frown on plaintext passwords and passphrases being written to configuration files or code. As such, we prefer to write the configuration based on the raw network key (computed using a function called _PBKDF_ in conjunction with _SHA1_).

!!! faq "Using `wpa_passphrase` to generate a raw PSK"
    You can generate the raw psk directly on your Pi by running the `wpa_passphrase` utility. This utility takes your SSID as an argument and then prompts you to enter your passphrase. 

    ```bash
    # Pass your SSID as the first argument
    wpa_passphrase "Home Wifi"
    ```
    
    You will not see any characters or placeholders echoed as you type the passphrase, but **`wpa_passphrase`** will continue to accept input until you hit _Enter/Return_. The output will be a valid **wpa_supplicant** configuration that you can paste into your configuration.
    
    ```
    network={
        ssid="Home Wifi"
        psk=2508539ff867a3578f6ba7d9ee1d4a62aea82c25d30ffb1eb3a05cd08a373c02
    }
    ```

## WPA2 Enterprise Networks

Unlike home and coffee shop networks, enterprise networks like _[eduroam](https://eduroam.org/)_, require a bit more setup since they authenticate individual users to the network as part of the process of establishing an encrypted connection. As such, these networks can provide substantially more security than networks that are protected by **WPA2 Personal**. They're also much more flexible. For example, once you have properly configured _eduroam_ to connect on your campus, you will also be able to get online at other schools offering _eduroam_ hotspots.

### Certificate-based Authentication

Enterprise networks utilizing certificate-based authentication rely on EAP-TLS mode of operation. Public key authentication has many security benefits over password-based mechanisms, though the process of onboarding new users and devices is more complex. Users on these networks are required to register or enroll in order to obtain a client identity certificate. 

Before you attempt to add an EAP-TLS network to wpa_supplicant, you must complete the onboarding process and install certificates onto your device. For University of Washington students, faculty, and staff, we have documented onboarding and certificate installation in separately _[(click here)](/resources/uw-wireless/#eduroam-onboarding){target=_blank}_.

!!! info "Eduroam configuration template (EAP-TLS)"
    The majority of the values needed for this template will be provided by your organization during onboarding.
    
    For **<ENCRYPTION_MODES>**, your organization most likely supports **CCMP**, which is a mode of **AES** encryption. If you're unsure, enter **CCMP TKIP** to allow for legacy encryption support (not recommended).

    All path names will be dependent on the installation location for your certificates.

    ```
    network={
        priority=10
        ssid="eduroam"
        scan_ssid=1
        key_mgmt=WPA-EAP
        proto=RSN
        pairwise=<ENCRYPTION_MODES>
        group=<ENCRYPTION_MODES>
        eap=TLS
        identity="<IDENTITY>"
        anonymous_identity="<ANONYMOUS_IDENTITY>"
        ca_cert="</PATH/TO/CA_CERT.PEM>"
        client_cert="</PATH/TO/CLIENT_CERT.PEM>"
        private_key="</PATH/TO/PRIVATE_KEY.PEM>"
        domain_match="<DOMAIN>"
        phase2="auth=<INNER_EAP_TYPE>"
    }
    ```

### Password-based Authentication

Password-based authentication can be set up for an enterprise wireless network by leveraging the Protected Extensible Authentication Protocol (PEAP) and the Microsoft Challenge Authentication Protocol version 2 (MSCHAPv2) to authenticate network clients based on a previously registered username and passphrase.

The following template configures wpa_supplicant for PEAP/MSCHAPv2 authentication on an Eduroam network. Add it to your **`wpa_supplicant.conf`**, substituting your own username and password hash for the supplied values.

??? danger "Avoid Credential Exposure"
    If implemented poorly, PEAP can expose your password to a malicious network operator. To prevent attacks, include the Certificate Authority (CA) certificate[^cacert] in your wpa_supplicant configuration. Without this information, the wireless client may send password information to an evil twin (malicious network posing as one you trust). 

    Likewise, we advise against hard-coding your plaintext school or work password into the wireless configuration file. Instead, compute an MD4 hash from your password and substitute it for the password. MD4 is a weak hash and fairly vulnerable to password cracking, but combined with a strong password, it does provide a layer of protection if your device is lost or stolen. 

    Follow these commands in order to compute the MD4 hash in Linux. The _history_ commands are not needed to compute a hash but are added for security. Without them, your password will be stored in the Bash history file and easily readable to anyone with access to your memory card.

    ```bash
    set +o history
    
    echo -n 'This is your password' | iconv -t utf16le | openssl md4
    # You should see output like 6f9bad2c90b80bd549e595fc91e27806
    
    set -o history
    ```

??? info "Eduroam configuration template (EAP-PEAP)"
    **This configuration is no longer supported on University of Washington's eduroam network (effective Feb 1, 2022).**

    ```
    network={
            ssid="eduroam"
            scan_ssid=1
            key_mgmt=WPA-EAP
            proto=RSN
            eap=PEAP
            identity="<USERNAME>@<ORGANIZATION_DOMAIN>"
            password=hash:6f9bad2c90b80bd549e595fc91e27806
            ca_cert="</PATH/TO/CA_CERT.PEM>"
            phase1="peaplabel=0"
            phase2="auth=MSCHAPV2"
    }
    ```

[^cacert]: Obtain this file from your organization's IT department.

## Applying Configuration Changes

Like other services, **wpa_supplicant** will not load our changes automatically. Rather than reset the daemon completely using **`systemctl`**, we can use **`wpa_cli`** to update the configuration and perform other basic maintenance.

When running **`wpa_cli`** we need to specify the interface we are configuring and a command to send to the **wpa_supplicant** service.

!!! instructions "Load configuration from `wpa_supplicant.conf`"
    ```
	# Update configuration from disk
	wpa_cli -i wlan0 reconfigure
	
	# Check the status of current connection
	wpa_cli -i wlan0 status
	```

See **`man wpa_cli`** for further instructions and examples.

