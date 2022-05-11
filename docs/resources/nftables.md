# Introduction to netfilter and nftables (2022-04-02)
This guide introduces nftables by building a basic ruleset that allows us to safely forward packets between internal and external networks while also masquerading traffic with the IP address of the external network interface. 

_Read the entire guide carefully before attempting to install the template or any of the other tools referenced here_.

## What is nftables
_nftables_ is the default tool used to manage netfilter's *packet filter rules* in Raspberry Pi OS. Though _nftables_ was originally released in 2014, it did not make its way into Raspberry Pi OS until the Bullseye release in November 2021. _nftables_ is the successor to _iptables_, which has been in use within the Linux community for more than 20 years.

_nftables_ is a packet filtering framework included in a variety of modern Linux implementations. _Packet filtering_ is a powerful technique that enables administrators to analyze and control the flow of inbound and outbound network traffic from any endpoint or intermediate network node. This level of control is essential for host- and network-based firewalls, network address translation (NAT), and other applications.

Packet filters define rules to match traffic based primarily on Layer 2 - 4 properties, such as: the inbound or outbound interface, type and length of messages, addresses, ports, and connection state. Each filter also describes actions taken when a match is found, e.g., log, deliver, or drop the packet. netfilter-based packet filters can also be used to apply NAT, make decisions about Quality of Service, or to reroute packets mid-flight.

_nftables_ builds on _netfilter_, an open-source project adding hooks, connection tracking, and other capabilities to the Linux network stack. These components enable dynamic filters, like _nftables_ rules, to be applied to every packet traversing the network stack. They are the secret sauce to many of the applications described above. 

## Tables, Chains, and Rule Syntax
In _nftables_, rules are composed of _expressions_ that are used to select packets and _statements_ that determine what actions _netfilter_ will perform when a match occurs. While rules are the meat of our filter configurations, _nftables_ doesn't allow them to stand alone. And so, before we explore the rule syntax, we need to looks _tables_ and _chains_. 

### Tables
_nftables_ uses a basic structure known as a _table_ to organize rules. Each _table_ is associated with a single _address family_ and includes rules to apply to packets of that family. 

This guide is focused on layer-3 traffic and considers tables for the following three address families:

- `ip`: for IPv4 rules (default)
- `ip6`: for IPv6 rules
- `inet`: for rules that apply to both IPv4 and IPv6 packets

While its predecessor, _iptables_, included a default set of tables for common packet filter functionality, _nftables_ administrators are left to define their own tables. For this exercise, we'll create an IPv4 `nat` table to implement _address translation_ between private and public addresses and an IPv4/IPv6 `filter` table to protect the Pi and the rest of our LAN from malicious traffic.[^table_names]

!!! example "Example: Empty _filter_ and _nat_ tables"
    ``` { .yaml .annotate }
    table inet filter {
        # Define chains (and rules) for IPv4 or IPv6 packets
    }

    table ip nat {
        # Define chains (and rules) for IPv4 packets
    }
    ```

[^table_names]: These names were chosen to mirror the built-in tables of _iptables_.

### Chains
Within each table, rules are grouped in _chains_. A _chain_ is an ordered list of rules describing the packets to match and the action to take when a match is found. When a chain is processed, the rules in it are tested in order from the beginning until a match is found. If _nftables_ finds a match while evaluating a rule, the remaining rules in that chain may be skipped depending on what action is taken.

#### Base Chains
_nftables_ won't evaluate the rules in a chain automatically. Rather, a chain will only be processed if it is explicitly called from a rule in another chain or if the chain has been associated with a [_netfilter hook_](/resources/netfilter-hooks).A chain that is associated with a hook is called a _base chain_.

Within the `filter` table, we are going to work with three base chains named `input`, `output`, and `forward`. These chains will be associated with _filter_ hooks of the same names[^chain_names], causing them to be processed when packets are being sent to software running on the Pi (`input` hook), when packets are being sent out by software on the Pi (`output`) hook, or when packets are being routed by the Pi from one network to another (`forward` hook). 

!!! example "Example: Filter table and empty base chains"
    ``` { .yaml .annotate }
    table inet filter {
        chain input {
            type filter hook input priority 0;
            # Rules are evaluated when a packet is intended for this host
        }
        chain forward {
            type filter hook forward priority 0;
            # Rules are evaluated when an incoming packet will be forwarded by this host
        }
        chain output {
            type filter hook output priority 0;
            # Rules are evaluated when a packet is being sent from this host
        }
    }
    ```

The `nat` table will have one base chain that is named for its respective _hook_, i.e., `postrouting`. Within netfilter, the postrouting hooks are the last ones to be processed when sending or forwarding a packet. As such, rules processed here can perform masquerading on the source IP addresses.[^prerouting]

!!! example "Example: NAT with postrouting base chain"
    ```
    table ip nat {
        chain postrouting {
            type nat hook postrouting priority 100;
            # Rules are evaluated after forwarding decisions in order to support source NAT or masquerading
        }
    }
    ```

[^chain_names]: This naming convention is influenced by _iptables_ and its built-in chains.

[^prerouting]: _netfilter_ also defines a `prerouting` nat hook that executes before routing decisions and can be used to implement port-forwarding and other destination NAT configurations.

### Rules Syntax
The basic _nftables_ rule syntax includes an expression and a statement. Expressions define conditions that need to be satisfied before an action is applied to a packet. Statements define the actions that will be taken. Since actions are the simpler component of a rule, we'll begin our exploration there. 

#### Statements (actions)
_nftables_ statements are used to control packet flow, apply _NAT_ transformations, compute metrics, record logs, and modify the order of rule evaluation. While the list below is not exhaustive, it is sufficient for this guide and the beginning firewall developer.

- [`accept`](https://wiki.nftables.org/wiki-nftables/index.php/Accepting_and_dropping_packets): Stop processing chain and deliver packet
- [`drop`](https://wiki.nftables.org/wiki-nftables/index.php/Accepting_and_dropping_packets): Stop processing chain and drop packet
- [`counter`](https://wiki.nftables.org/wiki-nftables/index.php/Counters): Increment byte and packet counters and continue processing chain
- [`log`](https://wiki.nftables.org/wiki-nftables/index.php/Logging_traffic): Write information about packet to logs and continue processing chain
- [`masquerade`](https://wiki.nftables.org/wiki-nftables/index.php/Performing_Network_Address_Translation_(NAT)#Masquerading): Replace source IP address with the address of the output interface and stop processing chain

You may observe that actions like `drop` and `accept` cause _nftables_ to stop processing the current chain. These actions are known as terminal actions. Actions like `log` and `counter` are non-terminal because they allow _nftables_ to continue processing even if a match had been found. _nftables_ supports multiple actions in a single statement, but a terminal may only appear as the final component of a rule. 

For example, we can produce a rule that increments a counter and delivers a packet by ending our rule with `counter accept`. We can also write a rule that increments a counter and logs a packet without specifying a policy decision by ending the rule with `counter log` or `log counter`. However, we cannot combine terminals or include a terminal before a non-terminal as with `accept counter` or `drop accept`.

#### Expressions (conditions)

Our rules need to describe the packets we want to filter or manipulate. In this exercise, our _primary consideration_ will be the _direction_ in which a given packet is moving through the Pi. This is particularly of importance for routing decisions and NAT. We can describe our criteria by examining the ingress network for incoming packets and/or the egress network interface for outgoing packets. 

!!! attention
    Throughout the instructions, we'll refer to the internal interface as our **&lt;LAN&gt;** (because it serves our local network) and the external interface as the **&lt;WAN&gt;** (because it connects us to the Internet). Your configuration files will reflect the system assigned interface names, such as: **`eth0`** and **`wlan0`**.

    It's up to you to determine, based on your learning so far, which interface corresponds to which placeholder.

Let's look at a rule that we can use to apply address translation to outbound packets. Since outbound packets will leave on the WAN, we can specify this restriction using the _outbound interface name (oifname)_ condition, i.e., `oifname <WAN>`. The following example masquerades outbound traffic on the **&lt;WAN&gt;** interface.

!!! example "Example: Applying NAT transformation to outbound traffic"
    ``` { .yaml .annotate }
    table ip nat {
        chain postrouting {
            type nat hook postrouting priority srcnat;
            oifname <WAN> masquerade
        }
    }
    ```

We can be even more specific with our rules by writing an expression with multiple conditionals. To identify outbound traffic forwarded from the LAN to the WAN, we can check both the inbound and outbound interface names by writing `iifname <LAN> oifname <WAN>`. The following example demonstrates a rule that explicitly accepts traffic flowing from **&lt;LAN&gt;** to **&lt;WAN&gt;**.

!!! example "Example: Combining multiple expressions"
    ``` { .yaml .annotate }
    table inet filter {
        chain forward {
            type filter hook forward priority 0;
            iifname <LAN> oifname <WAN> accept
        }
    }
    ```

##### Using Connection State in Rules
Netfilter is a _stateful_ filtering framework, meaning that it is even possible to specify nftables rules based on the state of a network connection for a given packet. This feature shapes our rules in two ways. 

First, you'll notice that we only had to specify the masquerade rule in the egress direction. The same rule also handles the de-masquerading process for ingress packets. 

Second, you'll see that we can base filtering decisions on connection state itself. While it's appropriate for NAT routers to allow outgoing traffic, we typically want to place restrict incoming traffic. When configuring a gateway connection for a home or office, we often block incoming traffic unless it relates to existing connections from the LAN. Unsolicited traffic does not make sense in this context and poses a security risk.

To accomplish this feat, we will make use of netfilter's connection tracking extensions and restrict incoming traffic to packets that are part of an established connection or related to a recent egress packet. The connection tracking portion of this expression is `ct state established,related`. 

!!! example "Example: Using the connection tracking extension"
    ``` { .yaml .annotate } 
        table inet filter {
            chain forward {
                type filter hook forward priority 0;
                iifname <WAN> ct state established,related accept
            }
        }
    ```

#### Default Policy (for Base Chains)
We are missing one piece to complete our configuration. By default, _nftables_ accepts a packet unless a match occurs on a terminal rule. Base chains can override this behavior by defining a default policy. We'll use this feature to create a firewall that fail securely, i.e., blocking traffic that we don't explicitly allow.

The following example adds a base chain policy in order to drop forwarded packets that we didn't allow from another rule.

!!! example "Example: Default drop policy"

    ``` { .yaml .annotate }
    table inet filter {
        chain forward {
            type filter hook forward priority 0; policy drop;

            # Explicitly allow "safe" connections
            iifname <LAN> oifname <WAN> accept
            iifname <WAN> ct state established,related accept
        }
    }
    ```

The _default deny_ posture is a best practice for your networks and devices, but it's important to approach the strategy with care. Without additional rules, attaching `policy drop` to the `input` and `output` chains will block inbound and outbound SSH and other protocols that might be needed for remote management. We will come back to this in a later guide, but we recommend for now that you leave the default accept policy on these chains.

### Final NAT Template
The following template draws together all of the components we have discussed so far to define a basic NAT configuration for a Linux router, leaving placeholder[^placeholder],[^quoting] elements for the reader to complete. Please review the previous examples and specifications that have been given in order to create a valid configuration. The following section will help you test and install your new ruleset.

!!! info Routing / NAT template
    ``` { .yaml .annotate }
    # Always flush the active ruleset before defining your new rules
    flush ruleset

    table inet filter {
        chain forward {
            type filter hook <HOOK> priority 0; policy <ACTION>;

            # Explicitly allow "safe" connections
            iifname <LAN> oifname <WAN> <ACTION>
            iifname <WAN> ct state established,related <ACTION>
        }

        chain input {
            type filter hook input priority 0;
            # You may leave this chain empty for now
        }

        chain output {
            type filter hook output priority 0;
            # You may leave this chain empty for now
        }
    }


    table ip nat {
        chain postrouting {
            type nat hook postrouting priority srcnat;
            oifname <WAN> masquerade
        }
    }
    ```

[^placeholder]: Text surrounded by &lt;&gt; symbols is indicative of a placeholder that should be filled in before loading the ruleset. 
[^quoting]: Do not quote text substituted for placeholders unless quotes are shown in the template.

## Configuring nftables
Throughout this guide, _nftables_ configuration has been presented in a declarative, table-based format that closely reflects the table/chain/rule hierarchy. _nftables_ also supports a command-based syntax that is passed to the _nft_ utility on the command line or from within scripts in order to create, read, update, and delete individual elements of a ruleset dynamically. 

We generally want firewall and routing rules to be configured as soon as possible during the boot process. When the _nftables_ service is started, _Raspberry Pi OS_ will initialize it with the contents of _/etc/nftables.conf_. 

While experimentation is usually encouraged, you may encounter negative consequences if you make haphazard modifications to this file. A [simple script](#editing-nftable-rules-safely) is provided below to support the learning process by testing your ruleset before making changes permanent.

### Enable the nftables Service
The _nftables_ service may not be running by default on your device. You can check on the current status of the daemon with `systemctl status nftables`. To launch the service immediately, run `systemctl start nftables`. To ensure that _nftables_ also starts automatically at boot, run `systemctl enable nftables`. 

### Editing nftable Rules Safely

!!! danger "Here be Dragons"
    Do not write rules directly to `/etc/nftables.conf`. Without proper testing, you risk creating a rule that locks you out of your device from the network.  

Create a working copy of `/etc/nftables.conf` into your _home directory_ _(it is okay to rename the file)_. Open the new copy and ensure that `flush ruleset` near the top (before your table definitions).[^flush] Proceed with any modifications to the working copy.

To test your ruleset, we recommend a scripted approach that can automatically revert a change that locks you out of the Pi. INFO314 students can find a copy of `nftables-apply.sh` in the project repository or create the script from the following listing.

??? info "nftables-apply.sh"
    ```
    #!/bin/bash

    # Adapted from https://sanjuroe.dev/nft-safe-reload (retrieved on 2022-04-03)
    # Modified to avoid editing system rules in-place

    SYSTEM_RULES="/etc/nftables.conf"
    TIMEOUT=45

    saved_rules=$(mktemp)
    cleanup() {
        rm -f "$saved_rules"
    }

    trap "cleanup" EXIT;

    # Waits $TIMEOUT seconds for a yes/no response
    read_approval() {
            read -t $TIMEOUT response 2> /dev/null

            case "$response" in
                    y|Y)
                            return 0
                            ;;
                    *)
                            return 1
                            ;;
            esac
    }

    # Make a copy of the active ruleset
    backup() {
            printf "flush ruleset\n" > $saved_rules
            nft list ruleset >> $saved_rules
    }

    # Apply a named ruleset
    apply() {
            nft -f "$1"
    }

    # Update system ruleset
    save() {
        local source_rules="$1"
        cp --no-preserve=mode,ownership "$source_rules" "$SYSTEM_RULES"
    }

    # Make a backup of the active ruleset in case of rollback
    backup
    new_rules="$1"

    if apply "$new_rules"; then
        printf "Are you still able to connect to your device (auto-rollback in $TIMEOUT seconds)? [y/n] "
            if read_approval; then
            save "$new_rules"
                    printf "\nUpdated system ruleset at ${SYSTEM_RULES}\n"
            exit 0
            else
                    apply "$saved_rules"
            printf "\nRolling back to original configuration\n"
                    exit 2
            fi
    fi
    ```

To test your rules, pass the name of your new rules file to the _nftables-apply.sh_ script, e.g., `./nftables-apply.sh nftables-test.conf`. If your rules load without any errors, you will be prompted to answer whether you can still connect. 

Open an _additional_ terminal window and launch SSH to connect to your Pi. If you are able to complete this step successfully, you can apply your changes to `/etc/nftables.conf` by answering `y` at the _nftables-apply_ prompt in your original session. 

![Accept New Rules](/img/accept-nft-rules.png)

If the test fails, answer **n** or wait 45 seconds for the rules to revert automatically.

![Reject New Rules](/img/rollback-nft-rules.png)

Once you are satisified that your rules are working correctly, it's a good idea to verify that your rules are loaded and that everything works correctly after a reboot. Use the `nft list ruleset` command to view active rules.

[^flush]: `nft -f <FILENAME>` is additive by default. Including `flush ruleset` before your table/chain definitions ensures that the new ruleset is properly loaded.

## Resources

- [Understanding Netfilter Hooks](/resources/netfilter-hooks)
- [Securing Your Server with nftables](https://www.datapacket.com/blog/securing-your-server-with-nftables)
- [Safe Reload with nftables](https://sanjuroe.dev/nft-safe-reload)
- [Linux Network Administrators Guide - Masquerading](https://www.oreilly.com/openbook/linag2/book/ch11.html)