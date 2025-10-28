## Introduction

~~While you can provision your own EC2-based DNS service solutions, AWS provides Route53 as an alternative. Route53 is Amazon’s global DNS service. Its primary purpose is to translate human-readable domain names into IP addresses. Specifically, It's an authoritative name server system that:~~
- ~~provides us the ability to make direct updates to manage public DNS names.~~
- ~~answers DNS queries, i.e. translates domain names into IP addresses associated with our AWS constructs/infrastructure (EC2, ELB, CloudFront distribution, etc.).~~

~~Route 53 is both a registrar and a DNS hosting provider. Route 53 is a domain registrar for hundreds of top level domains. If you have an existing domain name with another registrar, you can transfer it to Route 53.~~

## Route53 Aliases

CNAMEs and Route53 aliases:
- Similarities:
    - Both resolve DNS queries using other DNS records.
- Differences:
    - CNAMEs
        - has been around the longest and are a DNS standard record type, they are unofficially referred to as an "alias".
        - redirects requests to other DNS FQDNs, IP addresses are never supplied as the record value.
        - returns FQDN to client request, client must then perform further DNS queries to resolve the returned value (until it gets an IP address).
        - charge per-query.
        - The name of a CNAME record cannot be re-used within a zone (even by records with another types). This is actually not DNS standard, just a defacto law of DNS admins.
        - CNAME is not used to represent zone apex (means zone apex can't be used as the name of the CNAME record).
    - Route53 aliases:
        - not a standard DNS record type, it's for now an Route53 extension of functionality.
        - redirects requests to AWS service object FQDNs.
        - is recursively queries by Route53 until an actual IP address has been obtained and returned to client request.
        - no query charges.
        - usually has A/AAAA as record type and value as an AWS service object FQDN or other records within the same hosted zone.
        - The name of Route53 alias records can be re-used in valid ways within a same zone.
        - Alias record can be used to represent zone apex (means using zone apex as the name of the record).

## Hosted Zones & Zone Records

A Hosted Zone is a container that holds information about how you want to route traffic for a domain and its subdomains, just like a zone file for a particular DNS zone on traditional DNS servers. Hosted Zone contains Zone Records which are equivalent to resource records in a zone file.

2 types of Hosted Zones:
- public zone: handle DNS requests made over the public internet.
- private zone: handle DNS requests for private domains, need to be associated with >= 1 VPCs before they are used.
    - The VPC must have both the DNS support and DNS host names enabled within their VPC settings.
    - Since private hosted zones are not visible from the public internet, their domain names do not need to be registered with ICANN.
    - Private zones can't be a delegated subdomain of a public zone.
    - Private zoens can't delegate their own subdomains.

When you register a new domain name using Route53, then a public hosted zone for that domain namespace will automatically be created by AWS.

Once a hosted zone has been created, you automatically see 2 types of records:
- SOA: provides information about how your zone is configured.
- NS: identifies DNS servers that are authoritative for your zone.

Route53's record structure: record name, type, value, TTL and routing policy.

Use case: clients to be able to access a website hosted on a EC2 instance within AWS by querying `www.example.com` hosted on a Route53 public hosted zone:
- Simplest and least elegant solution: create an A/AAAA record within our hosted zone mapping `www.example.com` to the public IP address assigned to the EC2 instance. Problem is EC2 instances do not retain public IP addresses after stoppage.
- (If you only need 1 EC2 instance for your website) Resolve `www.example.com` to an Elastic IP address's public IP address associated to the EC2 instance using A/AAAA type.
- (If you host your website on a fleet of EC2 instances for scalability) Resolve `www.example.com` to an Elastic Load Balancer's FQDN using CNAME type. Problem:
    - charges are incurred by Route53 for CNAME records per query (though it can be small).
    - only the ELB's FQDN is returned to clients, thus clients need to spend more time querying Route53 to find a routable IP address for that FQDN.
- Since ELB is an AWS services; in our hosted zone we can create an alias with type of A and value of the ELB's FQDN.
    - no charges incurred since it's an alias record.
    - Route53 recursively perform queries until a routable IP address for the ELB's FQDN is found and return to client.
    - For example, a hosted zone config for `example.com.` might look like this:
    ```text
    www    A (alias)    ELB-hostname
    .      A (alias)    www.example.com     ; . represents the zone origin/hosted zone (`example.com.`)
    ```

## Routing Policy

Sometimes you want the value of a resource record to change dynamically to work around failures or ensure users get pointed to the least busy server. Route 53 lets you accomplish this with a variety of routing policies.

Note that all routing policies with the exception of Simple can use health checks to determine whether they should route users to a given resource, see [Health Checks](#health-checks).

### Simple

Simply lets you map a domain to a single static value (such as an IP address or logical resource), do not check whether the resource the record points to is available.

Simple routing policy can be extended to round robin like policy by creating multiple values for a single record, traffic will be evenly distributed across them.
```
// Hosted zone configuration in AWS env, not a zone file
beta.example.com      A            1.1.1.1 2.2.2.2
```

### Weighted

Just an extension of simple routing policy, in this policy you can have multiple records each with the same name.

Each record can be associated with a health check that checks the availability of the record value (e.g. IP addresss); and if the health check fails, the users are no longer routed to that failed record, the record and its associated weight will be taken out of the traffic distribution calculation.

### Failover

If primary resource is unavailable, redirect the traffic to secondary one.

You have to associate a health check to the primary resource to use with this policy because without one, the availability of the primary resource cannot be modified and no failover would happen.

Associating a health check to the secondary resource is optional.

In case both servers have associated health checks and they are reported as unhealthy, the primary server will always be returned.

### Gelocation

Lets you route users based on their specific continent, country or state.
- To be more exactly, it routes traffic based on the DNS resolver location that sends the request.
- To combat this, Amazon supports an extension of the DNS standard called `edns-client-subnet`, this allows the DNS resolver to forward a truncated version of the users' IP address to Route53 to help determine a more accurate location of the user.

### Multivalue Answer

Returns all hosts associated with the same record name.
```
// Hosted zone configuration in AWS env, not a zone file
www      A            1.1.1.1
www      A            2.2.2.2
// [1.1.1.1, 2.2.2.2] is returned when resolving www.beta.example.com
```

It supports return up to 8 hosts.

Each record can be associated with a health check that checks the availability of the record value; and if the health check fails, the value will not be included in the returned array.

### Latency

Sends users to resources in the AWS Region that’s "closest" to them (not Route53), i.e. has the lowest network latency as perceived by the AWS edge location that the users requests are being handled by. This means the latency is tested against the regional endpoints (e.g. edge location), not the value of Route53 records.

For example let say we have the following Latency routing policy config:
```
www.example.com   A  Latency-policy   1.1.1.1;  // 1.1.1.1 points to a resource in US West
www.example.com   A  Latency-policy   2.2.2.2;  // 2.2.2.2 points to a resource in Middle East
www.example.com   A  Latency-policy   3.3.3.3;  // 3.3.3.3 points to a resource in SEA
```

If (the resource behind) 1.1.1.1 becomes unavailable, Route53 latency routing mechanism would be unaware of it (because as mentioned above the latency is tested against the regional endpoint, in this case the one in US West) and would still return 1.1.1.1 when responding to clients in NA for example. The only way for NA clients' requests to be resolved with a different record would be if the entire US West region had a higher latency than any of the other regions used by alternative records.

To avoid that issue, it is recommended associating each record used in Latency routing policy with an (IP-based) health check that checks for the availability of the record value. Now if (the resource behind) 1.1.1.1 fails its health check, Route53 will no longer use that record and return the value of the one with next lowest latency.

## Health Checks

3 different types of Route53 health checks:

### Endpoint

This type of health check tests response from a single endpoint, it is the most commonly used type.

An endpoint is defined by:
- IP address (v4 or v6) or FQDN (can be any host anywhere in the world).
- port at the endpoint the health checks will be sent to.
- protocol used for check (HTTP, HTTPS, TCP).

Endpoint health checks can also check if a file path is accessible if protocol is HTTP/HTTPS.

Endpoint health check procedure (not sure if this also applies to other health check typs):
- In a single health check, tests are performed by multiple health checkers in various AWS regions.
- Each health checker sends health check tests to the endpoint every 10 or 30 seconds depending on configuration.
- A health check test is considered a failure if it does not receive a response from the endpoint within a certain amount of time depending on the protocol being used (endpoint must response within 4 sec for HTTP/HTTPs, 10 sec for TCP).
- A health checker will consider the endpoint unhealthy after a configurable number of consecutive failed tests.
- Route53 (the health check for the enpoint as a whole) will consider the endpoint unhealthy if less than 18% of health checkers consider the endpoint healthy (this threshold cannot be modified).

Route53 health checks prevent clients from being sent to offline endpoints.

It is not necessary to associate health checks with records that serve as aliases, since the AWS objects that aliases point to (API Gateway API, NLB, VPC endpoint, etc.) are intrinsically redundant due to their implementation.

Gotchas:
- FQDN health checks only work with A record (IPv4).
- When a FQDN resolves to multiple IP addresses (e.g. by using Multivalue Answer routing policy), health checkers (for that FQDN)' tests will be distributed to all IP addresses behind that FQDN. So essentially the health check as a whole is checking for the health of multiple IP addresses (under a same FQDN) and in this case we might want to be notified even if 1 IP address is unhealthy. However the health check could still consider the FQDN healthy even if some IP addresses behind it are unhealthy which is a false negative for us, this happens when there still exist some healthy IP addresses behind the FQDN and the number of health checks sent to them account for at least 18% of total number of all heath checks.
    - Best practice when creating health checks for systems with redundant responses is to use IP-based health check, not FQDN-based.

### Health checks

Health checks that check other health checks.
- 1 parent health checks will observer 1 or more child health checks.
- Parent health check is healthy if at least X child checks are healthy.

### CloudWatch alarms

Health checks tied to CloudWatch alarms.
- This type of health checks will monitor the CloudWatch data stream that is being sent to a previously configured alarm.
- Is considered healthy if alarm status is ok.
 
### VPC Endpoints

VPC endpoint is a way to access a public AWS service from within VPC without crossing the VPC boundary.

2 types, choosing which one to use depends on which AWS service you want to access within your VPC:
- Interface endpoints:
    - Powered by AWS PrivateLink: a way to provide private connectivity between VPCs.
    - AWS PrivateLink creates endpoints using ENIs with private IP addresses.
    - These endpoints server as entry points for your VPC traffic.
    - Many services are supported.
- Gateway endpoints.
    - Gateway endpoints are set up as logical constructs that we can specify as a target in our route table.
    - S3 and DynamoDB are supported.