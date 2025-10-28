---
mindmap-plugin: basic
tags:
  - aws
  - iam
  - aws-sts
---

# IAM

## **6. IAM Identity**
- IAM Resources ==that can be authorized using IAM Policy==.
- This includes
	- **IAM user**
		- Represent a single principal that need access to your account.
		- It has ==permanent (long-term) security credentials for authentication==.
			- Email+Password combo
				- Used for console access.
			- Access key
				- Used for programmatic access.
		- When you create an IAM user, you can choose to allow console or programmatic access.
		- You can have up to ==5000 IAM Users per account==.
		- An IAM user can be the ==member of up to 10 groups==.
		- Use cases
			- Giving a single thing long-term access to your AWS account/resources.
				- "Single thing" is from the admin's perspective,
				- Since nothing can prevent the IAM user from being secretly shared.
		- Policy types attached
			- Identity-based policies.
	- **IAM role**
		- An IAM role is intended to be assumable by anyone who needs it.
			- Different from IAM user, which is uniquely associated with one single principal.
		- It provides ==temporary (short-term) security credentials for authentication==.
			- The credentials is provided when a principal assumes/becomes a role.
			- When a principal assumes an IAM Role, this has an "authentication" implication that the principal becomes something trusted by AWS, which is an IAM Role.
			- The principal behind an IAM Role, before assuming the role, is usually already authenticated to AWS by different mechanisms.
				- IAM User (in same or different account than the role)
				- AWS services or application (code) authenticated by AWS STS.
				- Federated User authenticated by an external identity provider compatible with SAML 2.0, or OpenID Connect, or a custom-built identity broker.
		- Use cases
			- Giving AWS access to an unknown number or multiple things.
				- E.g. federated users access
			- Giving temporary AWS access to IAM users.
			- Cross-AWS account access.
			- Cross-service access.
			- Managing temporary credentials for applications running on EC2 instances and making AWS API calls.
		- Policy types attached
			- Trust policies (Resource-based policies type)
				- Control which principals can assume (become) that role.
			- Permission policies (Identity-based policies type)
				- Control what the role (or, the principal that assumed the role) is allowed to do.
		- Role types
			- Service role
				- IAM roles assumed by AWS services to perform actions on your behalf
				- Types
					- Service role for an EC2 instance
						- A special type of service role assumed by an application running an EC2 instance to perform actions in your account
						- The role will be assigned to the instance on launched
						- Applications running on the instance can retrieve temporary security credentials and perform actions the role allows (by including the credentials in the API requests)
						- Note that you could also authenticate/authorize applications on the instance by creating an IAM user with access key then store the key directly on the instance. But doing so is a hassle, especially if you want to rotate the keys regularly.
					- Service-linked role
						- A special type of service role that linked to an AWS service
							- It is owned/tied to the service
							- Its permissions are predefined by the service
						- The service can assume the role to perform actions on your behalf
						- Creation
							- A service might automatically create or delete the role
							- It might allow you to CUD the role as part of its setup process
							- It might require that you use IAM to create or delete the role
	- **IAM group**
		- A collection of IAM users.
		- IAM groups have NO credentials on their own, you CAN'T login to IAM groups.
		- Nesting groups (groups within groups) is NOT possible.
		- There's a limit of ==300 groups per account that can be increased== by contacting support.
		- Use cases
			- Organizing IAM users.
			- Apply permissions for multiple users at a time.
		- Policy types attached
			- Identity-based policies.

## **8. IAM Policy**
- An authenticated principal can send requests to AWS.
	- Exceptions: some services (e.g. S3, STS) allows a few requests from anonymous users.
- But to successfully complete the request, the principal must be authorized, i.e. has access.
- Access is managed by creating policies and attaching them to IAM Identity or AWS resources.
- Policies are JSON documents that, when attached to an IAM Identity or AWS resource, define their permissions.
- Policy structure
	- Optional policy-wide information at the top of the JSON file.
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
		- Implicit Deny (This is by default)
- Policy types
	- **Identity-based policies**
		- Attached to an IAM Identity to grant permissions to it.
		- Types
			- **Managed policy**
				- Standalone policy that can apply to a group of IAM Users, IAM Roles, IAM Groups.
				- Standalone = policy has its own ARN.
				- Types
					- **AWS managed policy**
					- **Customer managed policy**
			- **Incline policy**
				- Policy directly aded to a single IAM Identity.
				- It is directly embedded into the identity and maintains a strict 1-to-1 relationship with it.
	- **Resource-based policies**
		- Inline attached to resources to grant permissions to the principal specified in the policy.
		- [Not all services support this type of policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html).
		- Examples
			- S3 service: policy attached to S3 bucket.
			- IAM service
				- **Trust policy**
					- The only resource-based policy supported in IAM service, which is attached to an IAM Role.
					- Trust policy defines which principals can assume the role.
			- etc.
	- **Permission boundaries**
	- **Organizations SCPs**
	- **ACLs**
	- **Session Policies**

## Cost
- IAM is free.

## **7. Access Key**
- A ==long-term credential== tied to an IAM user.
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
- IAM controls Who (authentication) can do What (authorization) in your AWS account.
- IAM is a global service.
- IAM is globally resilience
	- Means it can cope with failure of a large section of AWS infrastructure.

## **AWS Security Token Service (STS)**
- Introduction
	- STS is used to create and provide authenticated/trusted principals with ==temporary security credentials==
	- That temporary security credentials can be used to control access to AWS resources
	- Temporary security credentials
		- Work almost identically to long-term access key credentials, except:
		- They are short-term, can be configured to last anywhere from few mins to several hours
		- They are not stored with the user, but generated dynamically and provided to user on requested
		- Advantages (over long-term credentials)
			- Avoid the hassle of distributing/embedding long-term credential with applications
- How it works
	- Scenario: a principal wants to assume a role
	- The principal calls STS's AssumeRole API (`sts:AssumeRole`)
		- The principal is either allowed or denied to assume the role based on the role's Trust Policy
	- If the principal is allowed, STS reads the role's Permission Policy to generate temporary credentials
		- Temporary credentials = f(args) | args contain Permission Policy
	- The credentials are returned back to the principal
	- The principal uses the credentials to authorize their requests to AWS services
	- When the credentials expired, another AssumeRole call is needed to get new credentials

## **5. Federation**
- (The creation of) a trust relationship between AWS and an external identity provider.
	- OpenID Connect compatible identity providers
		- Login with Amazon, Meta, Google, etc.
	- Security Assertion Markup Language 2.0 compatible identity providers
		- Microsoft Active Directory Federation Services.
- When the trust relationship is configured, the user of the external identity provider is assigned to an IAM role with temporary credentials.
- **Federated user**
	- Existing identities from external identity providers established a trust relationship with AWS.

## **1. IAM Resource**
- Resources stored in IAM
- Can be added, edited, removed from IAM.
- This includes
	- IAM User
	- IAM Group
	- IAM Role
	- IAM Policy
	- Identity-provider object

## **2. IAM Entity**
- IAM Resources AWS uses for ==authentication==.
- This includes
	- IAM User
		- Provides permanent (long-term) credentials for authentication.
	- IAM Role
		- Provides temporary (short-term) credentials for authentication.

## **3. Principal**
- A principal is ==a person or code== that make a request for an action or operation on an AWS resource. They do so by using AWS root user or IAM Entities (they are the ones that behind them).
- Depending on the context, the term "principal" can also be used to refer to things that are authenticated and identifiable within AWS (AWS root user, IAM Identities, Federated User, AWS services, etc.).
	- **IAM Principal**
		- Limiting the list to IAM Identities

## **4. Request**
- When a principal tries to use the AWS Management Console, the AWS API, or the AWS CLI, that principal sends a request to AWS.
- The request includes
	- **Actions or operations**
	- **Resources**
		- The AWS resource object upon which the actions/operations are performed.
			- A EC2 instance
			- An IAM user
			- An S3 bucket
			- etc.
	- **Principal**
		- The principal who sent the request, information includes the policies associated with it.
	- **Environment data**
		- IP address, user agent, SSL enabled status, timestamp, etc.
	- **Resource data**
		- Data related to the resource being requested (DynamoDB table name, EC2 instance tag, etc.)

## **AWS Organizations**