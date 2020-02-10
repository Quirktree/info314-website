# Introduction to Iptables
This guide serves the dual purpose of providing an introductino to iptables and using it to build a basic ruleset that routes packets between an internal and external network while also applying NAT to private, internal addresses. You should read the entire guide carefully before attempting to install the template or any of the other tools referenced here.

## Tables, Chains, and Rule Syntax

Iptables is a tool that has been used for more than 20 years to allow administrators to manage *packet filter rules* that shape the behavior of the Linux kernel's network stack.

### Tables
Iptables uses a basic structure called a table to group rules according to their main function. Common tables include `nat`, `filter`, `mangle`, and `raw`. For this exercise, we'll focus on the `nat` table to implement _address translation_ between private and public addresses along with the `filter` table to protect our network from outside traffic. 

### Chains
Within each table, rules are assigned to a structure called a chain. A chain is nothing more than an ordered list of rules. Rule conditions are tested in order from the beginning of the chain. Each rule ends with an action. If the conditions of a rule match, iptables will apply the specified action or jump to another chain without processing the other rules. If conditions do not match, the next rule in the chain will be evaluated.

The default chains correspond to different points in the packet lifecycle. Within the `filter` table, we will often work with the `INPUT`, `OUTPUT`, and `FORWARD` chains. These chains correspond to packets sent to the host (`INPUT`), packets being sent by the host (`OUTPUT`), and packets being routed through the host (`FORWARD`). 

Within the `nat` table, our focus will be on the `POSTROUTING` chain. This chain represents the last set of rules that will be evaluated before forwarding a filtered packet. This is where we want to apply address mappings related to SNAT and masquerading. In contrast, the `PREROUTING` chain is applied before any other process and is used to specify DNAT rules.

For a more detailed exploration of the chains we have discussed, see [Understanding Netfilter/Iptables Hooks](/resources/netfilter-hooks).

### Managing Rules from the Command Line
We can load a set of iptables rules from a file or manage them directly from the command line with the `iptables` command (`sudo` is required). As we explain the structure of rules, we will provide examples using the command line variant of iptables rule syntax.

The first component of a rule specified from the command line is the table name. Specify the table using the `-t` option to the `iptables command`. If you don't specify a table explicitly, the filter table will be used by default. 

When working with rules via the command line, we have to work with rules one at a time and specify the order of operations as we go. Recall from the last section that the order of rules in a chain is significant. We can append new rules to the end of a chain using the `-A` option or insert rules at the beginning of a chain using `-I`. Likewise, we can delete a rule from a chain by specifying `-D` and providing a numeric index to the rule.

??? attention "Copy/Paste Warning"
    These are examples only. Keep reading to learn how to build complete rules.

!!! example "Example: Appending Rules"
    Append a rule to the end of the POSTROUTING chain of the nat table ...

    ```
    iptables -t nat -A POSTROUTING <conditions> <actions>
    ```

!!! example "Example: Deleting Rules"
    Delete the first rule from the FORWARD chain of filter

    ```
    iptables -D FORWARD 1           
    ```

### Customize Rules with Filter Conditions

In addition to specifying table and chain, our rules need to describe the criteria associated with packets we want to filter or manipulate. In this exercise, our _primary consideration_ is the _direction of traffic_ for a given packet, which we describe by specifying the ingress and/or egress interface for a given packet. 

!!! attention
    Throughout the instructions, we'll refer to the internal interface as our **&lt;LAN&gt;** (because it serves our local network) and the external interface as the **&lt;WAN&gt;** (because it connects us to the Internet). Your configuration files will reflect the Raspbian-assigned interface names, such as: **`eth0`** and **`wlan0`**.

    It's up to you to determine, based on your learning so far, which interface corresponds to which placeholder.

For example, we want to apply address translation to outbound packets, i.e., packets being forwarded in the **&lt;LAN&gt;** interface and out the **&lt;WAN&gt;** interface. We can specify this restriction in our `nat` rule by providing the `-o` option with the name of the outbound interface. Likewise, we can instruct iptables to look at packets coming from the **&lt;LAN&gt;** and to the **&lt;WAN&gt;** by combining the `-i` option with `-o` in our filter rules.

??? attention "Copy/Paste Warning"
    These are examples only. Keep reading to learn how to build complete rules.

!!! example "Example: Adding filter conditions to rules"
    This NAT rule will be applied on outbound traffic that's being routed to the WAN network.

    ```
    iptables -t NAT -A POSTROUTING -o <WAN> <actions>
    ```

!!! example "Example: Combining multiple filters"
    The following filter rule will be applied to outbound traffic that's being forwarded from the LAN to the WAN.

    ```
    iptables -A FORWARD -i <LAN> -o <WAN> <actions>
    ```

### Using Connection State in Rules
Iptables is _stateful_, meaning that it is even possible to specify rules based on the relationship of a packet to other packets that were processed by iptables. This feature shapes our rules in two ways. 

First, you'll notice that we only have to specify the masquerade rule in the outward direction. The same rule also handles the de-masquerading process for incoming packets. 

Second, you'll see that we can base filtering decisions on connection state itself. While it's appropriate for NAT routers to allow outgoing traffic, we typically want to place greater restrictions on incoming traffic. When configuring a gateway connection for a home or office, we often block incoming traffic unless it relates to existing connections from the LAN. Unsolicited traffic does not make sense in this context and poses a security risk.

To accomplish this feat, we will make use of netfilter's connection tracking extensions which are specified with the `-m conntrack` and `--ctstate RELATED,ESTABLISHED` options.

!!! attention "Copy/Paste Warning"
    These are examples only. Keep reading to learn how to build complete rules.

!!! example "Example: Using the connection tracking extension"
    Check the connection state of ingress traffic.

    ```
    iptables -A FORWARD -i <WAN> -m conntrack --ctstate RELATED,ESTABLISHED <action>
    ```

### Specify an Action
The last component of an iptables rule is specified using the `-j` option. In a simple configuration, this option provides the action to be carried out against the packet after the match. While we will encounter the `MASQUERADE` option in our NAT rules, we will more frequently with basic `ACCEPT` and `DROP` rules within the filter context. 

In more complex configurations, `-j` can point to user-defined chains containing additional rules that relate to the pattern you just matched. As you gain experience, you'll see that this provides a greater degree of organization and control flow. 

Bringing this option together with the earlier examples, we can express complete rules using the following syntax:

```
iptables -t nat -A POSTROUTING -o <WAN> -j MASQUERADE
iptables -A FORWARD -i <WAN> -o <LAN> -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

### Default Actions
We haven't yet said what will happen if your iptables chains complete without any matching rules. Each iptables chain defines a policy that specifies the default actions that will be taken if no other rule matches in a chain.

Default policies are commonly used with the filter chains to implement a posture of failing securely, blocking traffic that wasn't explicitly allowed.

```
# Don't forward packets unless there was an explicit rule match
iptables -P FORWARD DROP  
```

Be careful, however, that you don't lock yourself out by a bad combination of rules and policies. For example, setting `-P INPUT DROP` and `-P OUTPUT DROP` will block all inbound and outbound traffic to your Pi unless you previously added rules to explicitly allow SSH or other tools that are needed for management purposes. Even when you correctly add these rules, it's relatively easy to lock yourself out of a device by flushing your current ruleset --  deleting the rules from all chains while leaving your policies intact.

### Final NAT Template
The following template draws together all of the components we have discussed so far to define a basic NAT configuration for a Linux router. The syntax for this template differs somewhat from the commands we explored above. Rather than invoking each rule or policy in this file through the `iptables` command, this template would be applied via `iptables-restore` or `iptables-apply`. 

To prepare to use this template, save a copy of this template to the home directory (file name does not matter) of your Linux router. Pay close attention to the comments within the file, since they explain some of the differences in syntax.

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
For simplicity, we’ll rely on on a collection of tools called `iptables-persistent` to test our rules and then load them automatically at boot. 

!!! danger "Here be Dragons"
    Do not write rules directly to `/etc/iptables/rules.v4`. If you do this, you risk creating a rule that locks you out of your device from the network. Without an external monitor and keyboard, you will not be able to bypass these changes since they are loaded at boot. 
    
    We are presenting `iptables-persistent` to provide you with a tool that allows us you to apply new rules safely by testing changes before making them fully persistent.

Like many other tools, `iptables-persistent` is available as a package in Debian's package repository. This package contains tools to help you test new rules and install them into the `/etc/iptables/rules.v4` path so that they can be loaded automatically in the future. Use `apt` to install the `iptables-persistent` package before attempting the instructions below.

Make a copy of your rules within your _home directory_ (file name does not matter) and use the `iptables-apply` command as shown below to test that they work before writing them to `/etc/iptables/rules.v4`. While it is possible to edit `/etc/iptables/rules.v4` directly, we recomend against it. 

The following command will flush iptables and load the rules given in `~/untested_rules`. After loading the rules, `iptables-apply` waits for **60 seconds** while you confirm that you can still SSH into the device from a second terminal tab or window. If your rules work as intended, you need respond affirmatively at this final prompt within the time limit so that iptables-apply can write them to a permanent location. If you skip this step, iptables-apply will assume that your test failed and restore the old version of the rules.

```
sudo iptables-apply -t 60 -w /etc/iptables/rules.v4 ~/untested_rules
```

After running the  command, you’ll have one minute to check that everything is working and confirm that you’re ready to apply the rules. If everything checks out, the rules will be written to `/etc/iptables/rules.v4` (where `iptables-persistent` can load them at boot). If your connection is interrupted or you decide not to keep the rules, the rules will be reset when the timer expires so that you con continue to access your Pi.

As a final test that your rules are loaded, reboot the Pi, connect with SSH, and list the rules with `sudo iptables-save`. If you don’t see your rules, something went wrong.

## Resources

- [Understanding Netfilter/Iptables Hooks](/resources/netfilter-hooks)
- [Digital Ocean Tutorial - Iptables and Netfilter Deep Dive](https://www.digitalocean.com/community/tutorials/a-deep-dive-into-iptables-and-netfilter-architecture)
- [Linux Network Administrators Guide - Masquerading](http://www.oreilly.com/openbook/linag2/book/ch11.html)
- [DigitalOcean Tutorial - Manipulate Iptables Rules](https://www.digitalocean.com/community/tutorials/how-to-list-and-delete-iptables-firewall-rules)
