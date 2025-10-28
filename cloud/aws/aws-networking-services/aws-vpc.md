---
mindmap-plugin: basic
tags:
  - aws
  - aws-vpc
---

# AWS Virtual Private Cloud

## Introduction
- A VPC is a Virtual Network inside AWS.
- VPC is a regional service
	- A VPC is within 1 account & 1 region.
	- A VPC spans all AZs in a single region.
- Sub title

## Types of VPC inside a region
- **Default VPC**
	- Each region has 1 default VPC, it can be deleted and recreated.
	- Private CIDR block range is always `172.31.0.0/16` and can't be changed.
	- It comes with 1 public [[aws-subnet]] in each AZ, an [[aws-igw]], [[aws-security-group]] and [[aws-nacl]].
	- It's best practice NOT to use the default VPC.
- **Custom VPC**
	- Each region can have multiple custom VPCs.
	- Custom VPCs are private and isolated (from other VPCs and internet) by default.