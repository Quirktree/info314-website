# Authoritative Name Server (last edited 2022-05-10)

## Overview
In this assignment, we will extend our BIND configuration (think back to our [_DNS Resolver checkpoint_](/assignments/resolver-setup/)) to handle DNS queries for a domain that is in your complete control. While you don't have very many services or resources available on your network yet, this project should help you explore the ways in which DNS can be used to identify resources on a network as you add them. This task will build upon the work you did in the [_DNS Planning Exercise_](https://canvas.uw.edu/courses/1355759/assignments/5195249).

!!! important
    With each checkpoint, we expect you to do more of the work on your own. While the initial Pi setup gave clear instructions, we are slowly moving toward giving you specifications and additional context that you need in order to determine the right steps to accomplish the task.

    In this assignment, you will use the provided reference documents to help you configure the project according to specification.

    For some students, this can be a challenging adjustment, but it is also a valuable skill to learn and practice as you prepare for technical internships and jobs.

## Before you start
Before you begin, make sure that you have completed all steps through Checkpoint #4 successfully. At this point, BIND is installed and configured so that your Pi can resolve DNS queries from clients inside your LAN. DHCP is configured to provide LAN clients with the IP address of this resolver (via the domain-name-server option), and you have tested that DNS resolution is working.

Before you begin, you should also confirm that you have completed the _LAN Planning_ and _DNS Planning_ exercises. You will be required to refer back to these worksheets.

### Configure the Name Server's IP Address

During our initial LAN planning, you selected a separate static IP address for your authoritative name server. Before starting on the application level configuration, be sure to configure the new address on the LAN interface.

## Create a new zone file
The documentation installed with BIND provides guidance for where to store zone files on your name server. The recommendation for primary name servers, such as the one we are creating, is to store the zone files in `/etc/bind` or one of its subdirectories. 

For this project, create a new sub-directory called `zones` within `/etc/bind` as shown below and copy the existing `db.local` into it as a starting point for creating your zone. Each domain used in this assignment will require its own zone file, named according to the following convention, `db.<domain_name>`. For example, the full path to the zone file storing gradebook.pi would be `/etc/bind/zones/db.gradebook.pi.`

!!! instructions "Creating your starter zone file"
    If you have copied db.local, the initial contents of your zone file will look similar to this:

    ```
    ;
    ; BIND data file for local loopback interface
    ;
    $TTL	604800
    @	IN	SOA	localhost. root.localhost. (
                    2		; Serial
                604800		; Refresh
                86400		; Retry
                2419200		; Expire
                604800 )	; Negative Cache TTL
    ;
    @	IN	NS	localhost.
    @	IN	A	127.0.0.1
    @	IN	AAAA	::1
    ```

    As is stated in the comment at the start of the file, this file is defining a special purpose zone for the localhost domain to ensure proper handling when the name is passed to BIND.

## Customize the localhost zone file for your own domains
Your task at this point is to update this file to describe the domains that you selected in the **DNS Planning Exercise**. 

!!! important
    Your primary resource for this task is the [Zone File Reference](/resources/zone-file-format), which outlines the format of the zone file and the _resource records_ that we will use. It may also be helpful to examine the section _Creating the Forward Zone File_ in [DigitalOcean's Tutorial](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-debian-9#configuring-the-primary-dns-server)

### Customize the Start of Authority (SOA)

Update the comments at the start of the file to refer to your domain and then modify the SOA resource record based on the fully-qualified domain name of your Pi and a valid email address, e.g., your UW email.

!!! important "Specifying e-mail addresses for SOA records"
    While it's not apparent on first glance, every SOA resource record contains a field called the RNAME, which specifies an email address associated with the administrator for the domain.
    
    As mentioned in [Zone File Reference](/resources/zone-file-format), RNAME records receive special formatting. You must replace the `@` with a `.`. If the user portion of the address contains any periods, prefix with an `\` to escape them.

### Add NS and A records for the nameservers
Delete the initial NS and glue records and create a new NS record and a type A glue record for your name server. The NS record will point to your server by its fully-qualified domain name. Likewise, the A record will contain the internal IP address of your server (as determined in LAN planning).

The [Zone File Reference](/resources/zone-file-format) provides real world examples of name servers and glue records. Using _dig_, e.g., `dig NS microsoft.com`, you can find additional examples for pretty much any public domain. 

### Add records for additional DNS resources
Using the provided [Zone File Reference](/resources/zone-file-format), create additional MX and A records to identify the mail server that you defined in the _DNS Planning Exercise_. 

It's okay that we haven't set up the mail server yet. Refer back to your _LAN Planning_ and select an unused static address. 

!!! attention
    Pay close attention to the expected format of an MX resource record. It varies slightly from other common record types and leads to frequent errors.

## Declare both zones in named.conf.local
The zone file that you have created defines the resources that can be queried by your name server, but it does not yet instruct BIND to serve the domain. To accomplish this, we will add a new `zone` configuration for each within `/etc/bind/named.conf.local`.

!!! instructions
    Open `/etc/bind/named.conf.local` and add a new zone reference:

    ```
    // Replace references to gradebook.pi with your own domain name
    zone "gradebook.pi" IN {
        //  We're setting up the primary server for the zone. In DNS, a primary 
        //  contains the zone file for a domain while secondary servers read the 
        //  zone contents from a primary server.
        type primary;
        file "/etc/bind/zones/db.gradebook.pi";
    };
    ```

## Update `listen-on` addresses
In an earlier step, you set up the layer-3 configuration for a new IP address on the Pi. Update the `listen-on` setting you defined in the previous checkpoint to include the name server address. 

With this change, BIND will be able to answer recursive and authoritative queries on both addresses. We'll make further changes in the final group project to add more security and limit which queries can be answered on each IP. 

## Validate and test your configuration
Run `named-checkconf -z` to identify any immediate errors in your configuration or zone file and restart the `bind9` service using `systemctl`.

Use `systemctl` to verify that the service is running correctly. You can also test your configuration locally on the Pi by running `dig @127.0.0.1 ns <DOMAIN_NAME>`. Pay close attention to status codes and errors appearing in the response. 

## Update DHCP
Your LAN clients should be able to query the new domain without any additional configuration; however, we will take the opportunity to make one additional update to DHCP. 

Following the same process you've encountered in previous guides add the `domain-name` option to your subnet configuration, providing the _private domain name_ establised in the DNS planning exercise.

This option configures your LAN clients to recognize your internal domain as a default domain name when attempting to resolve hosts, e.g., `dig titan` would resolve records for `titan.corp.gradebook.pi`.

## Additional Resources
* [A Comparison of DNS Server Types](https://www.digitalocean.com/community/tutorials/a-comparison-of-dns-server-types-how-to-choose-the-right-dns-configuration)
* [RFC 1034: Domain Names - Concepts and Facilities](https://tools.ietf.org/html/rfc1034)
* [RFC 1035: Domain Names - Implementation and Specification](https://tools.ietf.org/html/rfc1035)
* [How to format a zone file (Dyn.com)](https://help.dyn.com/how-to-format-a-zone-file/)
* [How to configure BIND as a Private Network DNS Server - DigitalOcean Tutorial](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-debian-9)
