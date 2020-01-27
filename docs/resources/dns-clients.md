# Managing DNS clients

## View or clear your local DNS cache
As applications make DNS queries to obtain the IP addresses of remote resources, your operating system will start to maintain a cache of previous responses. These cached responses are used on subsequent lookups in order to reduce network overhead and speed up the process of loading the applicable resources.

Clearing your DNS cache is an operating system dependent operation:

### Windows 10
Open PowerShell _as an administrator_ and run `clear-dnsclientcache`.

### macOS 10.11+: 
Run `sudo killall -HUP mDNSResponder` from Terminal[^sudo].

[^sudo]: Note that we are using `sudo` in order to perform this operation with root privileges.

### Linux
As an open source operating system with that comes in a variety of flavors, Linux users may find that some research is necessary to determine how DNS is managed in their distribution of choice and whether the system maintains a cache that can be cleared.

Linux DNS caches may be incorporated into a resolver (systemd-resolved), provided by a standalone service (nscd), or built into a name server (bind9) running on localhost.

A few of the most common options are listed below, along with the relevant command to restart the service and/or clear the cache directly.

Service | Description | Command
--- | --- | ---
systemd-resolved | DNS Resolver (distributed with systemd) | `sudo systemd-resolve --flush-caches`
nscd | DNS Cache | `sudo systemctl restart nscd`
dnsmasq | Name Server | `sudo systemctl restart dnsmasq`
bind9 | Name Server | `sudo systemctl restart bind9`
dns-clean | DNS Resolver (previously used by Ubuntu) | `/etc/init.d/dns-clean restart`


## Perform DNS lookups manually
At times it can be helpful to perform DNS queries manually. Tools like **`dig`** allow us to query a _nameserver_ and ask it to provide records about a particular host or domain. These tools provide us with an enormous amount of flexibility in interacting with DNS.

### Installing Dig
The **dig** command balances flexibility with ease of use, making it a popular tool for troubleshooting issues with DNS or performing security-related research on a domain. The utility is installed by default on macOS and some Linux distributions.

Instructions are provided below if dig isn't available on your system.

**Linux**

For Debian/Ubuntu based Linux including Arch, Mint, and Raspbian, **dig** is part of the _dnsutils_ package and is installed with `sudo apt install dnsutils`.

For Fedora/RedHat based Linux including CentOS, **dig** is part of the _bind-utils_ package and can be installed with `sudo dnf install bind-utils` -- use `sudo yum install bind-utils` if `dnf` is not available.

**Windows**

Windows users may install **dig** by downloading __ISC BIND 9__ and installing with the **Tools Only** option[^windows]. Similar functionality is also provided by the PowerShell **Resolve-DnsName** command. 

[^windows]: Detailed instructions provided at https://help.dyn.com/how-to-use-binds-dig-tool/.

### Common Usage
`dig <domain name>`: Request records for the given domain name. By default **dig** will send a query for **A** records (contain IPv4 addresses) to the default name server for the system.

`dig <type> <domain name>`: Override the type in order to obtain **cname** (DNS aliases for the given name), **mx** (mail servers), or **ns** name servers for the given host or domain rather than the default `A` record.

`dig @<name server IP> <domain name>`: Override the system's name server (often helpful to determine if other resolvers are returning different records).

!!! example
    ```
    pi@titan.local:~ $ dig @1.1.1.1 cname uw.edu

    ; <<>> DiG 9.10.6 <<>> @1.1.1.1 cname uw.edu
    ; (1 server found)
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50713
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 1452
    ;; QUESTION SECTION:
    ;uw.edu.				IN	CNAME

    ;; AUTHORITY SECTION:
    uw.edu.			600	IN	SOA	hanna.cac.washington.edu. domainmaster.cac.washington.edu. 2020012403 10800 1800 3600000 600

    ;; Query time: 40 msec
    ;; SERVER: 1.1.1.1#53(1.1.1.1)
    ;; WHEN: Sun Jan 26 23:32:59 PST 2020
    ;; MSG SIZE  rcvd: 105    
    ```

### Resolving DNS with PowerShell
PowerShell for Windows provides a powerful, scriptable DNS client that can be called via `Resolve-DnsName`. This tool replicates many of the features provided by dig, such as overriding the query type with the `-type <type>` option or the target server with `-server <name server IP>`.

Run `man Resolve-DnsName` from PowerShell or view the [online documentation](https://docs.microsoft.com/en-us/powershell/module/dnsclient/resolve-dnsname?view=winserver2012r2-ps) to learn more about this command. 
