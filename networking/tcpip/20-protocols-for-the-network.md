## DNS (Domain Name System)

### How it works

A client can ask to a DNS server (that is, a server that implements DNS protocol) the IP address of a website. Each computer is configured with the IP address of a DNS server, and it will contact that server for any DNS query it has to do.

DSN uses port **53:UDP** (server side) or **53:TCP** in case the traffic goes over a low-quality link or in case the response contains a lot of information.

Domain names are divided into **DNS Zones**, each zone managed by a set of servers (name servers). At the root of the Internet (*level-zero*), we have a zone named `.`, which is handled by the root DNS servers; we also have *first-level domain* or *top level domain* (like `com.`, handled by `.com` DNS zone), *second-level domain* (like `amazon.com.`, handled by `.amazon.com` DNS zone). n-level domain is the subdomain of (n-1) level domain.

A name server can be authoritative or non-authoritative:
- Authoritative means it provides answers to queries it knows about.
- Non-authoritative means it points to other servers or serves cached content from other name servers' data.

The name of a website or server, with all its parent domains (as an example, `google.com.vn.`) is called **Fully Qualified Domain Name (FQDN)**.

In real world, when a client a DNS server, that server may not know exactly what is the IP address of the website the client is looking for, but it knows about a more specific DNS managing this zone.
> For example, assume a client contacts the `.` (dot) DNS server asking for *mail.ictshore.com* (it's actually `mail.ictshore.com.` with the root level domain `.` at the end but we just don't see it):
> - `.` server knows `mail.ictshore.com` is part of the `.com` zone, so it redirects the client the IP address of a server managing the `.com` zone (for why `.` server knows about `.com` server, read [Subdomain Delegation](#subdomain-delegation)).
> - The client performs the same query to the `.com` DNS server, the server redirects the client the IP address of a server managing the `.ictshore.com` zone.
> - The client tries again, this time the `.ictshore.com` DNS server knows the IP address and responds the client.
>
> Another uncommon option is **Recursive DNS**, in which the DSN server performs other queries to other DNS servers instead of redirecting the client to the right zone. You should avoid this for 2 reasons:
> - An attacker can overwhelm the DNS server with a huge load of queries.
> - The server may take some time to perform all the queries, the client may timeout the connection.

Both DNS servers and clients store/cache DNS information, in the example above the client and all of the DNS servers first check locally to try to resolve address:
- the DNS client just remember the final binding between name and IP address, and keeps it for minutes or hours.
- the DNS server, instead, has a complex database that can be compared to a table (zone file) with several records of different types.

### Domain Registrar & DNS Hosting Provider

To ensure that no 2 entities try to use same (public) domain name, anyone who wants to have a public domain name must register it with a domain registrar. When registering a domain name, you must do so under a top-level domain (TLD): `.com`, `.net`, `.org`, etc.

It’s important to understand that domain name registration and DNS hosting are 2 different functions. Registering a domain gives you control over it for the duration of the lease, including the right to specify the service you want to provide DNS hosting for the domain. This means the domain registrar and DNS hosting provider don’t have to be the same company, but they often are.

### Apex Domain (Zone Apex)

An apex domain is the root of a registrable domain and does not contain a subdomain part, e.g. (`example.com`). Apex domains are often referred to as base domains, bare domains, naked domains, root apex domains, or zone apex domains.

> For example you purchased `example.com`, this will be the starting point of all further subdomains that relate to it, such as `www.example.com` and `beta.example.com`.

Apex domains are often securely redirected to a `www` subdomain, thus requiring an SSL certificate so visitors don't see security warnings in their browsers.

When redirecting an apex domain (where the apex domain is the record name) you typically need to use a DNS A/AAAA record, whereas subdomains should use a CNAME record. This is because an apex domain name is usually configured in A/AAAA record to be resolved to a specific IP address, thus using that same apex domain as an alias for another domain (configured in CNAME record) causes a logical confusion:
```text
; Invalid
example.com.  IN  CNAME  LB-Hostname
example.com.  IN  A      192.0.2.1
```

### Resource Record

Structure of a resource record:
- Record name: the domain name that you want to config, e.g. `example.com.`.
- Record type.
- Value: determined by record type.
- TTL: amount of time that non authoritative DNS servers will preserve information from this record within their cache.
- etc.

Some record types:
- `A`: maps a host to an IPv4 address.
- `AAAA`: maps a host to an IPv6 address.
- `CNAME`: Canonical Name records define an alias for a domain name.
- `NS`: Name Server records, used by other root-level domain servers resolve domain names within the zone. The name servers defined in NS typically receive their data from the primary name server.
- `MX`: Mail Exchange records are used to define mail servers.
- `TXT`: Text records hold text information, just a way for us to put unformatted or arbitrary text in a record.
- `PTR`: Pointer record is a reverse A record lookup. Maps an IP address to a host.
- `SOA`: Start of Authority.
    - Must have, define the primary name server for the domain, this name server maintains the master copy of the zone data and provides updates to other name servers as necessary.
- `CAA`: Certificate Authority Authorization.
- `NAPTR`: Name Authority Pointer.
- `SPF`: Send Policy Framework.
- `SRV`: Service locator.

### Zone file

A **zone file** is a text file that contains information about a particular DNS zone and its associated domain names, it defines the various resource records for the domain names, including information about the domain's name servers, IP addresses, and mail servers. An example of a zone file for the domain `example.com` on a name server:
```text
$ORIGIN example.com.                                   ; specify the zone origin
$TTL 3600                                              ; default exp time (in seconds) of all records without their own TTL value

; Domain names that end with . are already FQDNs.
; Must have, used to specify the primary authoritative name server (ns.example.com.) for the zone.
example.com.  IN      SOA       ns.example.com. username.example.com. ( 2020091025 7200 3600 1209600 3600 )  
example.com.  IN      NS        ns                     ; ns.example.com is an authoritative name server for example.com
example.com.  IN      NS        ns.somewhere.example.  ; ns.somewhere.example is another backup authoritative name server for example.com
example.com.  IN      MX        10 mail.example.com.   ; mail.example.com is the mailserver for example.com
@             IN      MX        20 mail2.example.com.  ; @ represents zone origin (example.com.), can be a different character (such as .) in other systems
example.com.  IN      A         192.0.2.1              ; IPv4 address for example.com

; Domain names that don't end with . are unqualified names and relative to the origin,
; they are combined with the zone origin to form FQDN, see $ORIGIN.
ns            IN      AAAA      2001:db8:10::2         ; IPv6 address for ns.example.com
www           IN      CNAME     example.com.           ; www.example.com is an alias for example.com
wwwtest       IN      CNAME     www                    ; wwwtest.example.com is an alias for www.example.com
```

When a public domain is registered with ICAAN, the corresponding zone file for that namespace will need to be created on at least one DNS server within the parent domain, this information is found in the NS record for the zone. In the example above, the zone file for `example.com` is created when `example.com` is registered, DNS (authoritative) servers that host the most up to date copy of the zone data is `ns.example.com` and `ns.somewhere.example`.

### Subdomain Delegation

To configure route traffic for subdomains such as `beta.example.com`, `public.beta.example.com`; technically, we can add records to the `example.com` zone (to specify how to route traffic for the subdomains) but that will not create true subdomains, i.e. these subdomains still reside within `example.com` zone file. A true subdomain has its own zone file with its own records that can be managed by a different group of admins, this allows separation of zone and record management within a larger DNS namespace.

So for example for `beta.example.com`, we should create a separate zone file for `beta.example.com` zone. DNS resolution process will still flow from the DNS global route `.` to the `.com` zone to the `.example.com` zone and eventually to the `beta.example.com` zone, to do that the parent `.example.com` zone file must contain NS records that resolve its respective subdomain (`beta.example.com` in this case) to `beta.example.com`'s authoritative servers (the same way that `.` zone contains `.com`'s and `.com` zone contains `.example.com`'s). After that, requests for subdomain zone `beta.example.com` can be forwarded by the parent zone `.example.com`, and subdomain zone `beta.example.com` still maintains separate management from the parent zone `.example.com`.

## DHCP (Dynamic Host Configuration Protocol)

DHCP is the protocol used to assign IP addresses to clients automatically, it uses **UDP port 68** on client and **UDP port 67** on server.

[If a client connects to a network](https://www.ictshore.com/wp-content/uploads/2017/01/1020-05-DHCP_lease_process.png):
- It first sends a broadcast **DHCP Discovery (D)** to see if there is a server in the network willing to give an IP address.
- Any **DHCP server** in the network that wants to give an IP address to the client replies with the a **DHCP Offer (O)**, containing an offered IP address.
- At this point, client officially requests the IP address offered by the server with a **DHCP Request (R)**.
- Then, the server pings that IP address with a **DHCP Acknowledgement (A)**. From now on, the client can use its new IP address.

> DHCP-obtained addresses do not last forever, they leased for a specific amount of time (generally 24 hours). In this time span, the DHCP server keeps track of the association of clients to IP addresses. To do that, it remembers the MAC address that issued the request.

> If more servers make an offer to the client, the client will make a Request for the Offer the client received first.

> Beside IP address, DHCP servers assign to clients a subnet mask and default gateway so that it will be able to interact in the network.

Suppose you are managing a network infrastructure for a large corporate environment, you can config a router so that it listen to DHCP requests from clients in a same subnet (this configuration is called **DHCP relay**), take them and put them in UDP segments destined to the remote HDCP server ([pic](https://www.ictshore.com/wp-content/uploads/2017/01/1020-06-DHCP_centralized_deployment_and_IPAM.png)).

## TFTP (Trivial File Transfer Protocol)

A very simple/trivial protocol to transfer file.

TFTP is mainly used to download logs or to push the operating system and/or the configuration to a remote network device, such as a switch or a router.

Unlike FTP, which works with two separate TCP channels (one for control, and one for data). TFTP implements control and data over a single channel, which is UDP-based and uses port 69 (server-side).

No secure version of TFTP currently exists.

## Dynamic Routing

[Read the original article](https://www.ictshore.com/free-ccna-course/protocols-for-the-network/)




