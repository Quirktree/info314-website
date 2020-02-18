# BIND Zone Files

A DNS zone file is a text file that describes a DNS zone and provides the list of all resource records that are defined in the zone. The zone file for an authoritative DNS server follows is composed of the following elements:

## Zone File Structure
* **Comments** beginning with a semicolon and continuing to the end of the current line.
* **Directives** such as `$ORIGIN` and `$TTL` keywords that provide default values used when interpreting DNS records.
* A **SOA** record positioned at the top of the file that defines global parameters for the zone.
* Additional **Resource Records** used to identify service-specific resources such as name servers and email servers or to map hostnames to network addresses.

The following simple zone file distributed with BIND9 defines a zone description for the reserved `localhost` namespace:

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

## Record Format

### Standard Fields
Each record in a zone defines a set of parameters associated with a single DNS resource.

* **NAME** is the DNS record's alphanumeric identifier (e.g., such as `www` or `mail`).
* **TTL** informs DNS clients, such as your your OS or browser, how long they can keep the record in their local cache. The TTL field is optional for individual resource records. BIND9 will default to the value of the global $TTL directive when omitted. 
* **CLASS** is a namespace identifier defined by the DNS protocol. For our purposes, the value will always be `IN`.
* **TYPE** describes the type of resource, e.g., NS (name server), A (address), or MX (mail server).
* **RDATA** specifies the value of the record. The format of this data is based on the record type. A few common types are given later in this document.

### Names
DNS names can be given in absolute or relative form. Absolute names are commonly called fully qualified domain name (FQDN) and include all parts of the name from the the host to the root of the DNS hierarchy. When configuring DNS servers, FQDNs are always terminated with a trailing dot, e.g., `www.washington.edu.`. 

Relative names omit the domain portion of the address and the trailing dot, e.g., `www`. When BIND encounters a hostname without the trailing dot in a zone file, it implicitly inserts the domain portion of the address. Forgetting to terminate an absolute hostname such as `www.washington.edu` with the trailing period will result in errors since BIND will see the resource as `www.washington.edu.washington.edu.`. 

### Shorthand
The standard zone file format employs a number of shortcuts that allow certain fields to be omitted or shortened. 

* BIND will use the global `$TTL` value as a default for records that do not specify their own.
* A hostname of `@` is shorthand for the default domain name for the zone. This value will be explicitly defined by the `$ORIGIN` directive at the start of a zone file or learned implicitly by the server based on the server's configuration. 
* When specifying multiple records in a row with the same name, you can omit the name after the first record.

## Common Record Types

### SOA Record Data
A valid zone file begins with the **Start of Authority** record. This record provides information about the primary servers for the domain and defines parameters to control the behavior of secondary servers.

RDATA for SOA includes the following fields:

* **MNAME** specifies the name server that is the primary source of data about this domain.
* **RNAME** is the email address of the administrator for the zone. RFC 2142 recommends that every domain provides a `hostmaster` mailbox to receive support queries related to the domain. **Special formatting applies for the RNAME (see below).
* **SERIAL** is an unsigned 32-bit version number of the zone-file used to detect updates. **This number must be incremented _every_ time the zone file is updated.**  RFC 1912 recommends using the numerically formatted date followed by a 2 digit revision number, i.e., `YYYYMMDDnn`. 
* **REFRESH** defines the number of seconds after which secondary name servers should query the master for the SOA record to determine whether there are any updates to the zone. RIPE's recommendation for small and stable zones is 86400 seconds (24 hours).
* **RETRY** defines the number of seconds a secondary name server should wait to retry its refresh query if the master does not respond. RIPE's recommendation for small and stable zones is 7200 seconds (2 hours).
* **EXPIRE** defines the number of seconds before expiring the current set of records if the master does not respond. RIPE's recommendation for small and stable zones is 3600000 seconds (1000 hours).
* **TTL** defines a time to live value that is used by recent servers to determine how long to cache a negative DNS result and by older servers to determine the default TTL for resources that don't specify it explicitly. RIPE's recommendation for small and stable zones is 172800 seconds (2 days).

> When formatting an email address for the RNAME field, You must escape any period in the username portion of the address with a `\` and replace the `@` symbol with an unescaped period. Do not escape periods in the domain portion of the address. An example can be seen in the following SOA record.

```
;
; Example SOA record for washington.edu
; Primary NS - hanna.cac.washington.edu
; Administrative Email - domainmaster@cac.washington.edu
;
washington.edu.		462	IN	SOA	hanna.cac.washington.edu. domainmaster.cac.washington.edu. 2019021600 3600 1800 3600000 600
;
; Use parenthesis to split the record across lines for readability
;
washington.edu 462 IN SOA hanna.cac.washington.edu. domainmaster.cac.washington.edu (
    2019021600          ; SERIAL
    3600                ; REFRESH (1 hour)
    1800                ; RETRY (30 minutes)
    3600000             ; EXPIRE (1000 hours)
    600                 ; TTL (10 minutes)
)
```

### NS Record Data
**Name Server** records are used to indicate the servers that host the zone file for a domain. A zone file will normally include its own name servers and the name servers of any subdomains. 

> In a real world scenario, your domains should be hosted by at least two name servers.

NS records refer to the hostname of a server. Assigning an IP address to a NS record is a misconfiguration. Rather a zone file can define glue records, A and AAAA address records that tie the name servers' hostnames to IP addresses. Glue records are required when a zone hosts its own authoritative name servers.

```
; Name Server
washington.edu.		86400	IN	NS	hanna.cac.washington.edu.

; Glue Records
hanna.cac.washington.edu. 86400	IN	A	140.142.5.5
hanna.cac.washington.edu. 86400	IN	AAAA	2607:4000:200:42::5
```

### A and AAAA Record Data
**Address** records map hostnames to IP addresses. The value of an A record in a BIND zone file is simply the IPv4 address in dotted decimal notation. For AAAA records, BIND supports the standard shorthand rules for IPv6.

It is legal (and common) for multiple address records to be provided for a given hostname. This feature allows us to support _dual stack_ hosts that can be reached using either IPv4 or IPv6 addresses, but it also enables us to distribute traffic between multiple servers for redundancy and load balancing. 

This configuration is referred to as _round-robin DNS_ and can be seen in the following example:

```
;
; Subsequent DNS responses will list records in a random order
;
www.washington.edu.	IN	A	128.95.155.134
www.washington.edu.	IN	A	128.95.155.197
www.washington.edu.     IN	A	128.95.155.198
```

### CNAME Record Data
**Canonical Name** records define aliases for hostnames. CNAME records have many uses, including mapping alternate domain registrations to a primary domain (as seen below) or to improve the stability of DNS records when we do not have direct control over the underlying infrastructure.

```
;
; www.uw.edu can be used as an alias for www.washington.edu
;
www.uw.edu.	IN	CNAME	www.washington.edu.
```

While CNAME records can be quite useful, there are several important restrictions to keep in mind:

* Do not define a CNAME record for the _apex domain name_, i.e., the name of the domain without any host prefix.
* Do not define a CNAME as the target of an MX or NS record. The target of an MX or NS must always resolve to an A or AAAA record.
* Do not define any other records for a hostname that is given a CNAME record. For example, since www.uw.edu has a CNAME, it cannot also be given an A record or a TXT record.

### MX Record Data
**Mail Exchanger** records are used to specify the servers that will receive mail for a given domain.

An individual MX record include an integer priority and a hostname. It is common and recommended to define multiple MX records for a domain. The priority field is used by mail agents to determine the preferred mail server (lower values are given higher priority).

```
;
; Since the three servers listed here have equal priority, a mail
; agent will choose one at random.
;
washington.edu.		10800	IN	MX	100 mxe30.s.uw.edu.
washington.edu.		10800	IN	MX	100 mxe31.s.uw.edu.
washington.edu.		10800	IN	MX	100 mxe32.s.uw.edu.

;
; The hostnames referred to by MX records must resolve to address 
; records are hosted in the zone for uw.edu, e.g.,
; 
mxe30.s.uw.edu.		86400	IN	A	173.250.227.19
```

### TXT Record Data
**Text** records hold freeform text that is generally intended for use by external services. 

TXT records are frequently used to:
* Verify ownership of a domain, e.g., required to set up 3rd party email hosting or obtain a certificate for TLS.
* Support SPAM reduction technologies such as the Sender Policy Framework (SPF) and DomainKeys Identfied Mail (DKIM).

```
;
; SPF uses TXT to indicate which networks or hosts may be used to send
; email on behalf of a domain. Recipients may flag or drop email that
; originated from an alternate server.
; 
uw.edu.			3600	IN	TXT	"v=spf1 ip4:128.95.242.222/32 ip4:128.208.0.5/32 ip4:128.208.181.0/26 ip4:140.142.32.0/24 ip4:140.142.234.128/25 ip4:173.250.227.0/24 ?all"
```

## References
* [RFC 1912: Common DNS Operational and Configuration Errors](https://tools.ietf.org/html/rfc1912)
* [RFC 2142: Mailbox Names for Common Services, Roles, and Functions](https://tools.ietf.org/html/rfc2142)
* [RIPE 203: Recommendations for DNS SOA Values](https://www.ripe.net/publications/docs/ripe-203)