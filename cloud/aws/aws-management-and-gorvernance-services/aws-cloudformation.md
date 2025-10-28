---
mindmap-plugin: basic
tags:
  - aws
  - cloudformation
---

# AWS CloudFormation

## Introduction
- An IaC tool specifically designed for AWS.
- It manages the lifecycle for your infrastructure
	- Creating and provisioning AWS infrastructures based on template/blueprint.
	- Updating infrastructure in a predictable and repeatable way.
	- Deleting infrastructure that you created easily, safely and reliably.
- How it works
	- The template is uploaded to CloudFormation service via S3, which kicks off CloudFormation orchestration.
	- CloudFormation confirms the actions it needs to take in CRUDing your infrastructure.
	- Then it orchestrates the API calls to various services to get the job done.

## Concepts
- #iac #infrastructure-as-code
	- First you start with a template that is an infrastructure blueprint you would like to provision in their *desired state*. This can leverage software development practices such as code reviews, version control and CI/CD
	- This template is run through some tooling which will parse the template and work out what needs to be done to the infrastructure to get into the desired state (e.g. CloudFormation, Terraform, Chef and Puppet).
	- The tooling will then make API calls to the targeted environment to provision the servers. It also handles changing infrastructure, which means it can calculate the delta between the current state and the desired end state.
	- The tooling thus enables you to focus on the template to reflect the desired end state rather than worrying about those delta changes.
- **Template**
	- Can be written in YAML or JSON format.
	- Top-level sections
		- **AWSTemplateFormatVersion**
			- Version of the CloudFormation template
		- **Description**
			- Description of the CloudFormation template.
			- Needs to directly follow the **AWSTemplateFormatVersion**
		- **Metadata**
			- Provides additional information about the template
		- **Parameters**
			- Allows you to parse values into your templates at runtime.
		- **Mappings**
			- Set of key value pairs which can operate as a lookup table
		- **Conditions**
			- A section that we can place conditions on whether certain resources are created
			- You can then use the conditions in **Resources** and **Outputs** sections
		- **Resources** (the only required section)
			- Defines the stack resources and their properties.

			-
			  ```yaml
			  Resources:
			  # This is NOT the physical ID which will be assigned to the resource
			  # on provision. Use the Physical ID to reference the resource outside the
			  # template and and Logical ID within the template.
			  LogicalID:
			  Type: "Resource Type", # e.g. AWS::EC2::Instance
			  Properties: # set of properties (optional and required)
			  ```

		- **Outputs**
			- Values that are returned whenever you view your stack's properties
- **Stack**
	- Stack is a set of related resources as a single unit.
	- When CloudFormation executes a template, it creates a stack containing the resources.
	- To update the resources within a template, you need to update a stack with a new (version of) template.
	- Stacks are created, updated, deleted in its entirety
		- If a stack cannot be created or updated in its entirety, CloudFormation will roll it back and automatically delete any resources that were created.
		- If a resource cannot be deleted, any remaining resources are retained until the stack can be successfully deleted.