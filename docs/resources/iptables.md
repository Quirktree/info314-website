# Introduction to Iptables

## Tables, Chains, and Rule Syntax

Iptables is a tool that has been used for more than 20 years to allow administrators to manage *packet filter rules* that shape the behavior of the Linux kernel's network stack.

### Tables
Iptables uses a basic structure called a table to group rules according to their main function. Common tables include `nat`, `filter`, `mangle`, and `raw`. For this exercise, we'll focus on the **`nat`** table to implement _address translation_ between private and public addresses along with the **`filter`** table to protect our network from outside traffic. 

### Chains
Within each table, rules are assigned to a structure called a chain. The order of rules in each chain matters. Once a rule is matched, iptables will jump to a specified action (or another chain) without processing the other rules. 

The default chains correspond to different points in the packet lifecycle. Within the `filter` table, we will often work with the `INPUT`, `OUTPUT`, and `FORWARD` chains. These chains correspond to packets sent to the node, packets being sent by the node, and packets being routed through the node respectively. 

Within the `nat` table, our focus will be on the `POSTROUTING` chain. This chain represents the last set of rules that will be evaluated before forwarding a filtered packet. This is where we want to apply address mappings related to SNAT and masquerading. In contrast, the `PREROUTING` chain is applied before any other process and is used to specify DNAT rules.

### Managing Rules from the Command Line
The `iptables` command allows you to manipulate rules directly from the command line (`sudo` is required). When you create a rule from the command line, specify the table using the `-t` option (filter is the default if you omit the option). 

We can append new rules to the end of a chain using the `-A` option or insert rules at the beginning of a chain using `-I`. Likewise, we can delete a rule from a chain by specifying `-D` and providing a numeric index to the rule.

??? attention "Copy/Paste Warning"
    These are examples only. Keep reading to learn how to build complete rules.

!!! example "Example: Appending Rules"
    Append a rule to the POSTROUTING chain of the nat table ...

    ```
    iptables -t nat -A POSTROUTING  
    ```

!!! example "Example: Deleting Rules"
    Delete the first rule from the FORWARD chain of filter

    ```
    iptables -D FORWARD 1           
    ```

### Customize Rules with Filter Conditions

In addition to specifying table and chain, our rules need to describe the criteria associated with packets we want to filter or manipulate. In this exercise, our _primary consideration_ is the _direction of traffic_ for a given packet. For example, we want to translate the address of packets being forwarded in the **&lt;LAN&gt;** and out the **&lt;WAN&gt;** interface. 

!!! attention
    Throughout the instructions, we'll refer to the wired interface as **&lt;LAN&gt;** (because it serves our local network) and the wireless interface as the **&lt;WAN&gt;** (because it connects us to the Internet). Your configuration files will reflect the Raspbian-assigned interface names, such as: **`eth0`** and **`wlan0`**.

    It's up to you to determine, based on your learning so far, which interface corresponds to which network.

We can specify this restriction in our `nat` rule by providing the `-o` option with the name of the corresponding interface. Likewise, we can instruct iptables to look at packets coming from the **&lt;LAN&gt;** and to the **&lt;WAN&gt;** by combining the `-i` option with `-o` in our filter rules.

??? attention "Copy/Paste Warning"
    These are examples only. Keep reading to learn how to build complete rules.

!!! example "Example: Adding filter conditions to rules"
    This NAT rule will be applied on traffic that's being routed to the WAN network.

    ```
    iptables -t NAT -A POSTROUTING -o <WAN>
    ```

!!! example "Example: Combining multiple filters"
    The following filter rule will be applied to traffic that's being forwarded from the LAN to the WAN.

    ```
    iptables -A FORWARD -i <LAN> -o <WAN>
    ```

### Using Connection State in Rules
Iptables is _stateful_, meaning that it is even possible to specify rules based on the relationship of one packet to others that have been received. This feature shapes our rules in two ways. 

First, you'll see that our masquerading rule also handles the de-masquerading process for incoming packets. 

Second, you'll see that we can base filtering decisions on connection state itself. While it's appropriate for NAT routers to allow outgoing traffic, we typically want to place greater restrictions on incoming traffic. When configuring a gateway connection for a home or office, we typically want to block incoming traffic unless it relates to existing connections from the LAN. 

To accomplish this feat, we will make use of netfilter's connection tracking extensions which are specified with the `-m conntrack` and `--ctstate RELATED,ESTABLISHED` options.

!!! attention "Copy/Paste Warning"
    These are examples only. Keep reading to learn how to build complete rules.

!!! example "Example: Using the connection tracking extension"
    Check the connection state of ingress traffic.

    ```
    iptables -A FORWARD -i <WAN> -m conntrack --ctstate RELATED,ESTABLISHED
    ```

### Specify an Action
The last component of an iptables rule is specified using the `-j` option. In a simple configuration, this option provides the action to be carried out against the packet after the match. In more complex configurations, `-j` can point to a user-defined chain containing additional rules. As you gain experience, you'll see that this provides a greater degree of organization and control flow. While we will encounter the `MASQUERADE` option in our NAT rules, we will more frequently with basic `ACCEPT` and `DROP` rules within the filter context. Bringing this option together with the earlier examples, we see that we can express complete rules using the following syntax:

```
iptables -t nat -A POSTROUTING -o <WAN> -j MASQUERADE
iptables -A FORWARD -i <WAN> -o <LAN> -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

### Default Actions
When you write iptables rules, keep in mind the default actions that will be taken if no other rule matches in a chain. Default actions are configurable through policies that are defined for each chain. From a security point-of-view, it's a good practice to set default policies to block traffic that wasn't explicitly allowed.

```
# Don't forward packets unless there was an explicit rule match
iptables -P FORWARD DROP  
```

Be careful, however, that you don't lock yourself out by a bad combination of rules and policies. For example, setting `-P INPUT DROP` and `-P OUTPUT DROP` will block all inbound and outbound traffic to your Pi if you don't already have rules set up to explicitly allow SSH or other tools.

### Final Template
The following template draws together all of the components we have discussed so far to define a basic NAT configuration. The syntax for this template varies slightly. Rather than invoking each rule or policy through the `iptables` command, this template would be applied via `iptables-restore` or `iptables-apply`.

```
# Fill in the parameters in <> in order to complete the ruleset

# Rules are given on a table-by-table basis, beginning with the *<table> and
# ending with COMMIT.

# Rules are appended to a chain using the -A <chain> syntax. Parameters vary
# somewhat by chain and type of effect desired.

# The -j switch instructs the rules engine to jump to another chain or perform
# the action on the matched packet. The engine will not evaluate any more rules
# in the given chain.

# The default policy for a chain is given by the lines beginning with
# :<chain>. The [0:0] resets the packet/byte counters that tell us how many
# times the default policy has been applied. Note that default policy is
# evaluated after all rules, so it only applies if there were not any matches.

*nat
-A POSTROUTING -o <interface> -j MASQUERADE
COMMIT

*filter
:FORWARD <action> [0:0]
-A FORWARD -i <interface> -o <interface> -j ACCEPT
-A FORWARD -i <interface> -o <interface> -m conntrack --ctstate RELATED,ESTABLISHED -j <action>
COMMIT
```

## Loading rules from the file system with `iptables-persistent`
For simplicity, we’ll rely on on a collection of tools called `iptables-persistent` to load the rules at boot. On boot, `iptables-persistent` will automatically load rules from `/etc/iptables/rules.v4`.

Like many other tools, `iptables-persistent` is available as a package in Debian's package repository. Use `apt` to install the `iptables-persistent` package before proceeding with the remaining steps.

Make a copy of your rules within your home directory and use `iptables-apply` to test that they work before writing them to `/etc/iptables/rules.v4`. While it is possible to edit `/etc/iptables/rules.v4` directly, it is not recommended.

!!! danger "Always test your rules"
    The risk of adding rules directly to `/etc/iptables/rules.v4` is that you create a rule that locks you out of your device and cannot bypass it since it is loaded at boot. `iptables-persistent` provides a tool that allows us to test a set of rules before making them fully persistent.

The following command will load the rules from `~/untested_rules` and wait for **60 seconds** while you confirm that you can still SSH into the device. If you respond affirmatively to the final prompt within the time limit, the rules will remain active **and** be written to a location where they are automatically reloaded at boot.

```
sudo iptables-apply -t 60 -w /etc/iptables/rules.v4 ~/untested_rules
```

After running the  command, you’ll have one minute to check that everything is working and confirm that you’re ready to apply the rules. If everything checks out, the rules will be written to `/etc/iptables/rules.v4` (where `iptables-persistent` can load them at boot). If your connection is interrupted or you decide not to keep the rules, the rules will be reset when the timer expires so that you con continue to access your Pi.

As a final test that your rules are loaded, reboot the Pi, connect with SSH, and list the rules with `sudo iptables-save`. If you don’t see your rules, something went wrong.

## Resources

- [Iptables and Netfilter Deep Dive](https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture)
- [Linux Network Administrators Guide - Masquerading](http://www.oreilly.com/openbook/linag2/book/ch11.html)
- [DigitalOcean Tutorial - Manipulate Iptables Rules](https://www.digitalocean.com/community/tutorials/how-to-list-and-delete-iptables-firewall-rules)
