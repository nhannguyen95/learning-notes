## Concepts

### AWS CDK

AWS Cloud Development Kit.

### AWS CDK app

An AWS CDK app is an application written in supported high-level programming languages that uses the AWS CDK to define AWS infrastructure.

An app defines one or more CloudFormation stacks.

Stacks contain constructs.

Each construct defines one or more concrete AWS resources, such as S3 buckets, Lambda functions, etc.

### Constructs

Constructs are the basic building blocks of AWS CDK apps. A construct represents a "cloud component" and encapsulates everything AWS CloudFormation needs to create the component.

A construct can represent:
- a single AWS resource, such as an S3 bucket.
- a higher-level abstraction consisting of multiple related AWS resources, such as a worker queue with its associated compute capacity.

Constructs are represented as classes in your programming language of choice. You instantiate constructs within a stack to declare them to AWS, and connect them to each other using well-defined interfaces.

### AWS Construct Library

AWS Construct Library contains constructs for every AWS service. The main CDK package is called `aws-cdk-lib`, and it contains the majority of the AWS Construct Library.

## AWS CDK Toolkit

Command line tool for interacting with CDK apps.

Developers can use the AWS CDK Toolkit to synthesize artifacts such as AWS CloudFormation templates and to deploy stacks to development AWS accounts.