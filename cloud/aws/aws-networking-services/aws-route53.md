---
mindmap-plugin: basic
tags:
  - aws
  - route53
---

# AWS Route53

## #Introduction
- While you can provision your own EC2-based DNS service solutions, AWS provides Route53 as an alternative
- Route53 is Amazon’s global (DNS) service and globally resilient.
- Main functions
	- Register domain names
		- Route53 acts as a #domain-registrar.
		- Route53 has relationships with hundreds of top-level domains.
		- If you have an existing domain name with another registrar, you can transfer it to Route53.
	- Route internet traffic to the resources for your domain
		- Route53 acts as a #dns-hosting-provider.
		- It answers DNS queries, i.e. translates domain names into IP addresses associated with our AWS constructs/infrastructure (EC2, ELB, CloudFront distribution, etc.).
	- Check the health of your resources.