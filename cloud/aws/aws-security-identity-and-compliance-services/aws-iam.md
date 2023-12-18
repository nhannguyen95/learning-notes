---
mindmap-plugin: basic
tags:
  - aws
  - iam
---

# IAM

## IAM identities
- #iam-user
	- Represent an identity (humans or applications) that need access to your account.
	- It has permanent long-term credentials and is used to directly interact with AWS services.
	- Use case
		- Give a single thing(a person, a laptop, etc.) access to your AWS account/resources.
	- You can have up to 5000 IAM Users per account.
	- An IAM User can be the member of up to 10 groups.
- #iam-group
	- A collection of related (IAM) users (dev team, finance, HR, etc.)
	- Helps you apply common access controls to all group members.
- #iam-role

## #iam-policy
- Policy is used to define permissions for them.
- By default, users/groups/roles can't do anything until you attach them with a policy (document).
- Policy types
	- Identity-based policies
		- Managed policy
			- Standalone policy that can apply to a group of identities (users, groups, roles).
			- Standalone = policy has its own ARN.
			- Managed policy types
				- AWS managed policy
				- Customer managed policy
		- Incline policy
			- Policy created for a single identity (a user, a group, a role)
			- It is directly embedded into the identity and maintains a strict 1-to-1 relationship with it.
	- Resource-based policies
	- Permission boundaries
	- Organizations SCPs
	- ACLs
	- Session Policies
- Policy Document
	- Most policy documents are stored in AWS as JSON formats.
	- A policy document includes
		- Optional policy-wide information at the top of the document
		- One or more individual statements.
			- Each statement includes information about a single permission.
			- Statement structure
				- `Sid` (optional)
					- Optional statement ID to differentiate statements.
				- `Effect`
					- Either `Allow` or `Deny`.
					- Takes effect when `Action` and `Resource` match the operation you're attempting to do.
				- `Action`
					- A list of actions that the policy allows or denies.
					- Example: `[s3:*]` means all actions to S3 services.
				- `Resource` (required only in some circumstances)
					- A list of resources to which the action apply.
					- Optional when creating a resource-based policy
						- If not provided, the resource to which the action applies is the one to which the policy is attached.
- Policy evaluation logic when there's conflicts (simple view)
	- Conflicts = multiple statements applied on same resource.
	- Decreasing-priority order
		- Explicit Deny
		- Explicit Allow
		- Implicit Deny

## Cost
- IAM is free.

## #iam-access-key
- A long-term credential tied to an IAM user (or the AWS account root user).
- Used for programmatic access to AWS services when typing CLI commands (without a username/password).
- An Access Key is constituted of 2 parts.
	- Access Key ID: IAM user identifier.
	- Secret Access Key: like a private key, used to digitally sign the payload.
- How it works (roughly)
	- When AWS CLI sends a request, payload is signed with the Secret Access Key by generating an HMAC.
	- The payload, HMAC, Access Key ID is sent to AWS.
	- AWS verifies the identity of the sender (using Access Key ID) and the integrity of the payload (by recomputing the HMAC).
	- If it's verified, the request is accepted and processed.
- The term "rotating access keys" simply means creating a brand new Access Key and remove the old one, that's why an IAM user can have (maximum) 2 Access Keys.
- Access Keys can be disabled/activated/deleted.

## Introduction
- IAM controls Who / What identities (authentication) can do What (authorization) in your AWS account.
- IAM is a global service.
- IAM is globally resilience
	- Means it can cope with failure of a large section of AWS infrastructure.