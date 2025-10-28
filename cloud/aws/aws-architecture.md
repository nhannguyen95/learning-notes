---

mindmap-plugin: basic

tags:

- aws

- aws-account

- aws-root-user

---

# AWS Architecture

## Network Zones Categorization
- **"Public Internet" Zone **
	- This is the (public) internet.
- AWS Zones
	- **"AWS Private" Zone**
		- AWS VPCs.
		- It can only access the internet (and the internet can only access it) if you allow it.
		- It can be configured to connect with on-premise networks.
	- **"AWS Public" Zone**
		- This *runs between* the internet and the AWS Private Zone.
		- It's not part of the internet, it's connected to the internet.
		- It can be thought as the internet-facing zone of AWS.
		- AWS public services (services with public endpoints like S3) operate from this zone.
		- Access
			- *Internet <-> AWS Public Zone*: When you access this zone with a public internet connection, the internet will be used for transit.
			- *Internet <-> AWS Private Zone*: It allows AWS Private Zone resources to access the public internet (as long as the resource has a public IP address), in this case it's like a bridge between AWS Private Zones and the public internet.
			- *AWS Private Zone <-> AWS Public Zone*: It allows AWS Private Zone resources to access AWS public services, note that in this case the connection does not touch the public internet at any point.

## Global Network
- #aws-region
	- An AWS region is an area selected by AWS, it does not map onto a continent or a country.
	- AWS regions are geographically spread, we can use it to design systems that withstand global level disasters.
	- Inside each region is a full deployment of AWS infrastructure (compute, storage, database, etc.)
	- Most AWS services are tied to a specific region. This means when you use an AWS service, high chance that you are interacting with them in a specific region.
	- Each region has multiple AZs located at a sufficient distance.
	- Number of AZs depend on the region, but each region has at least 2.
	- Implication when choosing a region
		- Geographic separation -> fault tolerance.
		- Geopolitical separation -> your system (data, etc.) affected by laws and regulations of (countries of) that region.
		- Location control -> closer to your users -> performance.
- #aws-edge-location
	- Much more smaller than regions.
	- Located at many more places than regions.
	- Where AWS deploys physical server infrastructure (such as CDNs) to provide low-latency access.
- #aws-availability-zone
	- A logically isolated data center in a region.
	- 1 AZ can have 1 or more physical data centers.
	- AZs are connected through low-latency, redundant links -> requests between different AZs (not necessarily in same region) aren't as expensive as requests across internet, in terms of latency.
	- Latency within an AZ (such as EC2 -> EC2 in the same subnet) is lower compared to latency across AZs.
	- AZ identifier is generated randomly for each AWS account to distribute resources across different AZs
		- E.g. `us-east-1a` in 2 different AWS accounts can point to 2 different AZs.
	- An AZ is associated with 1 single region, they don't span region.

## Service Resilience
- Global Resilient
	- IAM
	- Route53
- Region Resilient
- AZ Resilient

## Shared Responsibility Model
- AWS is responsible for the security of the cloud.
- Customer is responsible for the security in the cloud.

## #high-availability (HA), #fault-tolerance (FT), #disaster-recovery (DR)
- HA
	- HA is about maximizing uptime (minimizing downtime) of a system.
	- It's NOT about not having any failure/outage.
	- It's about fast/automatic recovery of issues.
	- HA is generally expressed in the form of percentage of uptime a year.
		- 99.9% (3 9s) = 8.77 hour/year downtime.
		- 99.999% (5 9s) = 5.26 min/year downtime.
- FT
	- FT is more than HA, more complex and expensive to implement than HA.
	- FT = HA + system still be operational through failure/outage (no downtime during recovery).
- DR
	- What to plan for/to do to recover a system before/after a natural or human-induced disaster knocks it out.
	- It's used when HA and FT don't work.

## Amazon Resource Name (ARN)
- Uniquely identify resources within any AWS accounts.
- Formats
	- `arn:partition:service:region:account-id:resource-id`
	- `arn:partition:service:region:account-id:resource-type/resource-id`
	- `arn:partition:service:region:account-id:resource-type:resource-id`