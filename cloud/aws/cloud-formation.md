## IoC

First you start with a template that is an infrastructure blueprint you would like to provision in their **desired state**. This can leverage software development practices such as code reviews, version control and CI/CD.

This template is run through some tooling which will parse the template and work out what needs to be done to the infrastructure to get into the desired state. Some examples of this tooling are CloudFormation, Terraform, Chef and Puppet.

The tooling will then make API calls to the targeted environment to provision the servers. It also handles changing infrastructure, which means it can calculate the delta between the current state and the desired end state. The tooling thus enables you to focus on the template to reflect the desired end state rather than worrying about those delta changes.

## CloudFormation

- It is a IoC tool for AWS, designed specifically for AWS.
- It manages the lifecycle of your infrastructure:
  - Creating and provisioning your AWS infrastructure based on the template/blueprint.
  - Updating your infrastructure in a predictable and repeatable fashion.
  - Deleting infrastructure that you created easily, safely and reliably.

CloudFormation template can be written in YAML or JSON format.

The template is uploaded to CloudFormation service via S3, which kicks off CloudFormation orchestration. CloudFormation confirms the actions it needs to take in CRUDing your infrastructure.

Then it orchestrates the API calls to various services to get the job done.

## Template

Some sections:
- Metadata (optional): section for objects that provide additional information about the template.
- Parameters (optional): allows you to parse values into your templates at runtime.
- Mappings (optional): set of key value pairs which can operate as a lookup table.
- Conditions (optional): a section that we can place conditions on whether certain resources are created.
- Transform (optional): where you can specify transforms that CloudFormation will use to process your template, for example including external snippets into a template.
- Resources (the only required section): defines the stack resources and their properties.
- Outputs (optional): allows you to return values from your stack.

Resources section format:
```yaml
Resources:
    LogicalID:  # this is not the physical ID which is actually assigned to the resource
                # on provision. Use the Physical ID to reference the resource outside the
                # template and and Logical ID within the template.
        Type: "Resource Type",
        Properties:  # set of properties (optional and required)
```

## Stack

- Stack is a set of related resources as a single unit.
- When CloudFormation executes a template, it creates a stack containing the resources.
- To update the resources within a template, you need to update a stack with a new (version of) template.

## Change Sets

Before updating a stack, you can generate a change set. A change set allows you to preview how the changes will impact your running resources, it is basically a diff between the current state of the stack and the proposed one.

This can be very important for live systems. For example, renaming a RDS instance will:
- create a new one.
- delete the old one.

this can result in a potential data loss.

4 operations of a Change Set:
- Create: create a change set of an existing stack by submitting a modified template, and/or parameter values. This does not modify existing stack.
- View: after creating, you can view proposed changes.
- Execute: executing the change set updates the changes on the existing stack.
- Delete: you can delete a change set without performing the changes.

Change Set contains the following changes information for a resource:
- Action: action taken on the resource (Add/Modify/Remove).
- LogicalResourceId & PhysicalResourceId (e.g. `EC2Instance` and `i-03248u4r98e4h423r`).
- ResourceType (e.g. `AWS::EC2::Instance`).
- Replacement: indicates if a resource will be replaced or simply modified in place. If true, CloudFormation will delete the old resource and create a new one. Replacement can be Conditional, meaning that the resouce might be replaced and can be determined by the template alone. We need to reference to documentations for more details.
- Scope: which resource attribute is triggering the update.
- Details: description of the change to the resource.


## CloudFormation Features

### Intrinsic Functions

Intrinsic functions are built-in functions that help you manage your stacks.

For example, `Join` function which appends a set of values into a single value:
```yaml
!Join [ ":", [ a, b, c ] ]  # "a:b:c"
```


### Pseudo Prameters

Pseudo parameters are predefined by CloudFormation and do not need to be declared in the template. They are similar to Environment Variables. They can be referenced with the `Ref` Intrinsic Function.
```yaml
Resources:
    Ec2Instance:
        Type: 'AWS::EC2::Instance'
        Properties:
            Tags:
                - Key: "Name"
                  Value: !Join
                    - ""
                    - - "EC2 Instance for "
                      - !Ref AWS::Region  # pseudo parameter
```

Some most important ones:
- `AWS::AccountId`: returns the AWS account ID of the account.
- `AWS::NotificationARNs`: returns the list of notification ARNs for the current stack.
- `AWS::StackId`: returns the ID of the stack.
- `AWS::StackName`: returns the name of the stack.
- `AWS::Region`: returns a string representing the AWS Region in which the resource is being created.

### Mappings

Mappings enable you to use an input value to determine another value.

```yaml
Mappings:
    RegionMap:  # logical name
        us-east-1:
            AMI: ami-1234
        us-west-1:
            AMI: ami-5678

Resources:
    Ec2Instance:
        Type: 'AWS::EC2::Instance'
        Properties:
            ImageId: !FindInMap  # Intrinsic Function
                - RegionMap           # 1st parameter - the logical map name to look up to
                - !Ref 'AWS::Region'  # 2nd parameter - the top level key
                - AMI                 # 3rd parameter - the second level key
```

### Input Parameters

Input Parameters enable us to input custom values to our template.
- Each parameter must be assigned a value at runtime, you can optionally specify a default value.
- The only required attribute is `Type` which is the data type.

Supported parameter types: `String`, `Number`, `List<Number>`, `CommaDelimitedList`, AWS-specific types (e.g. `AWS::EC2::Image::Id`), Systems Manager Parameter types.

```yaml
Parameters:
    InstTypeParam:
        Type: String
        Default: t2.micro
        AllowedValue:
            - t2.micro
            - m1.small
            - m1.large
        Description:
            EC2 Instance Type

Resources:
    Ec2Instance:
        Type: AWS::EC2::Instance
        Properties
            InstanceType:
                Ref:
                    InstTypeParam
```

You will be asked to provide the value for parameters when you feed the template into CloudFormation.

### Outputs

Outputs enable us to get access to information about resources within a stack. For example, ouputs the Public IP or DNS of an EC2 instance in the stack.

```yaml
Outputs:
    InstanceDns:  # Logical name of the output
        Description:
            The Instance Dns
        Value:
            !GetAtt
                - MyEC2Instance  # Logical name
                - PublicDnsName
```

### User Data

A resource of type `AWS::EC2::Instance` has a property called `UserData` which enables us to perform actions on system startup.
- `UserData` is Base64 encoded.
- Works for both Linux and Windows.
- Only runs on the first boot cycle.
- Beware of the time it takes to execute, it might input startup time for your instance.

`UserData` for Linux:
- Run as root (no need for sudo command).
- Not run interactively (no user feedback).
- Logs output to `/var/log/cloud-init-ouput.log`.
- Starts with `#!` and the interpreter.

```yaml
Resources:
    EC2Instance:
        Type: AWS::EC2::Instance
        Properties:
            UserData:
                !Base64 |  # The literal block allows us to input multiple lines of strings.
                    #!/bin/bash -xe
                    yum update -y
                    ...
```

Procedural scripting for `UserData` is not ideal cause the script can get messy very quickly. To solve this problem, CloudFormation provides Python based helper scripts to optimize this. The scripts come preinstalled on Amazon Linux. These scripts are not executed automatically, you need to call them when need. They can be updated manually using `yum install -y aws-cfn-bootstrap`.

4 CloudFormation helper scripts provided by AWS:
- `cfn-init`: use to retrieve and intepret resource metadata in `AWS::CloudFormation::Init`, install packages, create files and start services.
- `cfn-signal`: use to signal CloudFormation when resource or application is ready.
- `cfn-get-metadata`: use to retrieve metadata based on a specific key.
- `cfn-hup`: use to check for updates to metadata and execute custom hooks when changes are detected.

An example of using `cfn-init`:
```yaml
Resources:
    EC2Instance:
        Type: AWS::EC2::Instance
        Metadata:
            AWS::CloudFormation::Init:
                config:  # The default config set
                    # Followed by a list of cfn-init script's parameters.
                    packages:  # Specify the packages that we need to install..
                        yum:   # ..via yum repository.
                            httpd: []
                            php: []
                    files:     # Create files
                        /var/www/html/index.php:
                            content: !Sub |
                                <?php print "Hello world"; ?>
                    services:  # Start services
                        sysvinit:
                            httpd:
                                enabled: true
                                ensureRunning: true
        Properties:
            UserData:
                'Fn::Base64':
                    !Sub |  # Substitutes pseudo parameters into a literal block.
                        #!/bin/bash -xe
                        # Ensure AWS CFN Bootstrap is the latest
                        yum install -y aws-cfn-bootstrap
                        # Install the files and packages from the metadata of the 'EC2Instance' instance
                        /opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource EC2Instance --region ${AWS::Region}
```


