## Concepts

~~### IoC~~

~~First you start with a template that is an infrastructure blueprint you would like to provision in their **desired state**. This can leverage software development practices such as code reviews, version control and CI/CD.~~

~~This template is run through some tooling which will parse the template and work out what needs to be done to the infrastructure to get into the desired state. Some examples of this tooling are CloudFormation, Terraform, Chef and Puppet.~~

~~The tooling will then make API calls to the targeted environment to provision the servers. It also handles changing infrastructure, which means it can calculate the delta between the current state and the desired end state. The tooling thus enables you to focus on the template to reflect the desired end state rather than worrying about those delta changes.~~

~~### CloudFormation~~

- ~~It is a IoC tool for AWS, designed specifically for AWS.~~
- ~~It manages the lifecycle of your infrastructure:~~
  - ~~Creating and provisioning your AWS infrastructure based on the template/blueprint.~~
  - ~~Updating your infrastructure in a predictable and repeatable fashion.~~
  - ~~Deleting infrastructure that you created easily, safely and reliably.~~

~~CloudFormation template can be written in YAML or JSON format.~~

~~The template is uploaded to CloudFormation service via S3, which kicks off CloudFormation orchestration. CloudFormation confirms the actions it needs to take in CRUDing your infrastructure.~~

~~Then it orchestrates the API calls to various services to get the job done.~~

### Template

Top-level sections:
- ~~`AWSTemplateFormatVersion` (optional): version of the CloudFormation template, only accepted value is `2010-09-09`.~~
- ~~`Description` (optional): description of the CloudFormation template.~~
- ~~`Metadata` (optional): that provide additional information about the template.~~
- ~~`Parameters` (optional): allows you to parse values into your templates at runtime.~~
- `Rules` (optional): a set of rules to validate the parameters.
- ~~`Mappings` (optional): set of key value pairs which can operate as a lookup table.~~
- ~~`Conditions` (optional): a section that we can place conditions on whether certain resources are created. You can then use the conditions in Resources and Outputs sections.~~
- `Transform` (optional): where you can specify transforms that CloudFormation will use to process your template, for example including external snippets into a template.
- ~~`Resources` (the only required section): defines the stack resources and their properties.~~
- `Hooks` (optional): used for ECS Blue/Green deployments.
- ~~`Outputs` (optional): values that are returned whenever you view your stack's properties.~~

#### Parameters

Parameters enable us to input custom values to our template.
- Each parameter must be assigned a value at runtime, you can optionally specify a default value.
- The only required attribute is `Type` which is the data type.

Supported parameter types:
- String
- Number: an integer or float.
- List<Number>: an array of integers or floats.
- CommaDelimitedList: an array of literal strings.
- AWS-specific parameter types: AWS values such as Amazon VPC IDs (e.g. `AWS::EC2::VPC::Id`).
- SSM Paramter Types: parameters that correspond to existing parameters in Systems Manager Parameter Store (e.g. `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>`).

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

#### Mappings

Mappings enable you to use an input value to determine another value.

```yaml
Mappings:
    RegionMap:             # logical name of the map
        us-east-1:         # top level key
            AMI: ami-1234  # second level key
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

#### Conditions

To use conditions in your template, you specify the parameters you want your conditions to evaluate in Parameters section. Then you define your conditions using intrinsic conditions functions. Then in the Resource or Outputs sections, you associate conditions with resources/outouts that you want to conditionally create, or use the `Fn::If` intrinsic function to conditionally specify resource property values based on a condition you define.

```yaml
Parameters:
  EnvType:
    Description: Specify the Environment type of the stack.
    Type: String
    AllowedValues:
      - test
      - prod
    Default: test
    ConstraintDescription: Specify either test or prod.

Conditions:
  IsProduction: !Equals
    - !Ref EnvType
    - prod

Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !If [IsProduction, t2.small, t2.micro]  # conditions at properties level

  Volume:
    Type: AWS::EC2::Volume
    Properties:
      Size: 2
      AvailabilityZone: !GetAtt EC2Instance.AvailabilityZone
      Encrypted: true
    Condition: IsProduction
```

#### ~~Resources~~

Resources section format:
```yaml
Resources:
    LogicalID:  # this is not the physical ID which is actually assigned to the resource
                # on provision. Use the Physical ID to reference the resource outside the
                # template and and Logical ID within the template.
        Type: "Resource Type",
        Properties:  # set of properties (optional and required)
```

#### Outputs

Outputs enable us to get access to information about resources within a stack. For example, ouputs the Public IP or DNS of an EC2 instance in the stack.

Furthermore, output values can be imported into other stacks, these are known as cross-stack references.

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

### ~~Stack~~

- ~~Stack is a set of related resources as a single unit.~~
- ~~When CloudFormation executes a template, it creates a stack containing the resources.~~
- ~~To update the resources within a template, you need to update a stack with a new (version of) template.~~
- ~~Stacks are created, updated, deleted in its entirety:~~
    - ~~If a stack cannot be created or updated in its entirety, CloudFormation will roll it back and automatically delete any resources that were created.~~
    - ~~If a resource cannot be deleted, any remaining resources are retained until the stack can be successfully deleted.~~

### Nested Stacks

Nested Stacks are stacks created as part of other stacks using the `AWS::CloudFormation::Stack` resource.

Nested Stacks provide a mean to group common patterns into separated templates that could be reused/referenced in others.

Nested Stacks can themselves contain other nested stacks, resulting in a hierarchy of stacks. Certain stack operations, such as stack updates, should be initiated from the root stack rather than performed directly on nested stacks themselves.

Whilst single templates can be deployed from your local machine, Nested Stacks require that the nested templates are stored in an S3 bucket.

To reference a CloudFormation stack in your template, use the `AWS::CloudFormation::Stack` resource:

```yaml
Resources:
    NestedStack:
        Type: AWS::CloudFormation::Stack
        Properties:
            TemplateURL: 'Path/To/Template'
            Parameters:  # passing parameters to the nested stack.
                ExampleKey: ExampleValue
```

### Layered Stacks

Nested stacks enable you to reuse templates (classes) but not the created stacks (objects) themselves. To resue a stack, you can use [Exports](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html) to create global variables for that stack, then use [Imported](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html) to reference to it.

Each export must be unique per account and per region.

### Change Sets

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

Change sets don't indicate whether your stack update will be successful.

### Stack Sets

You can use Stack Sets to create, update, delete stacks across multiple accounts and AWS regions with a single operation. You will need admin role to perform Stack Sets operations, and an execution role to deploy the actual stacks in target account(s).

## CloudFormation Features

### Intrinsic Functions

Intrinsic functions are built-in functions that help you manage your stacks.

For example, `Join` function which appends a set of values into a single value:
```yaml
!Join [ ":", [ a, b, c ] ]  # "a:b:c"
```

Intrinsic functions can only be used in certain parts of a template: resources, outputs, metadata attributes, update policy attributes.

### Pseudo Parameters

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
- `AWS::Partition`: returns the name of the AWS partition. The partition name for standard AWS Regions is `aws`, for the China Region is `aws-cn`, for the AWS GovCloud Region is `aws-us-gov`.

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

### Helper Scripts

Procedural scripting for `UserData` is not ideal cause the script can get messy very quickly. To solve this problem, CloudFormation provides Python based helper scripts to optimize this. The scripts come preinstalled on Amazon Linux. These scripts are not executed automatically, you need to call them when need. They can be updated manually using `yum install -y aws-cfn-bootstrap`.

4 CloudFormation helper scripts provided by AWS:
- `cfn-init`: use to retrieve and intepret resource metadata in `AWS::CloudFormation::Init`, install packages, create files and start services.
- `cfn-signal`: a way to instruct AWS CloudFormation to complete stack creation only after all the services (such as Apache, `cfn-hup` are running and not after all stack resources are created).
- `cfn-get-metadata`: use to retrieve metadata based on a specific key.
- `cfn-hup`: enables existing EC2 instances to apply template updates of UserData (e.g. changes to updating software packages, configuring application settings, restarting services, etc.) without having to either replace the EC2 instance or manually apply the update outside of CloudFormation. When a CloudFormation stack is updated, the cfn-hup script is notified via an SNS topic and then triggers a set of actions defined in a configuration file. This configuration file specifies which files or commands should be executed to update the instance's configuration.

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

### Return values

To understand what available values returned when you use `Ref` or `Fn::GetAttr` intrinsic function on a resource of a specific type, reference [AWS resource and property types reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html).

For example:
- If you want to reference an S3 bucket name, use `Ref` followed by the logical ID of the bucket resource.
- If you want to retrieve the ARN of the bucket, use `Fn::GetAtt` with the `Arn` attribute.

The `Fn::Sub` intrinsic function can be used to retrieve values that `Ref` and `Fn::GetAtt` return for a specified resource using the same format of logical ID and return value attribute.

### Resource dependencies

There are some cases where you want to explicitly define resources creation order, you can use `DependsOn` attribute to do so (an implicit resources creation order is when you reference a resource in another one by using `Ref`, `Fn::GetAttr`, etc.):

```yaml
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      Tags:
        - Key: Name
          Value: Resource-dependencies-workshop

  SNSTopic:
    Type: AWS::SNS::Topic
    DependsOn: S3Bucket  # Without this, S3Bucket and SNSTopic will be created parallelly
    Properties:
      Tags:
        - Key: Name
          Value: Resource-dependencies-workshop
```

### Dynamic References

With dynamic references, you can reference external values stored in AWS services such as AWS Systems Manager, AWS Secrets Manager in your CloudFormation template.

```yaml
Resources:
    Instance:
        Type: AWS::EC2::Instance
        Properties:
            ImageId: '{{resolve:ssm:/golden-images/amazon-linux-2}}'  # reference value in AWS Systems Manager
```

### Packaging

In some cases, a CloudFormation template refers to other files or artifacts, for example a Lambda source code, zip file or nested CloudFormation files. It is cumbersome to upload those artifacts to S3 first before we can deploy the template. Instead, we can reference them as local files, then use `aws cloudformation package` to generate a new template whose references to those local files are replaced with S3 URIs.

### Stack policy and prevention controls

You can use CloudFormation features such as:
- [Stack Policy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/protect-stack-resources.html) to determine which update actions you can perform on resources you manage with your stack.
- [Termination Protection](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-protect-stacks.html) to prevent stack deletion.
- [DeletionPolicy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-deletionpolicy.html) to retain resources that you describe in your stack when you delete the stack.
- etc.

### Resource importing

You can [import](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resource-import.html) an existing resource into a (new or existing) CloudFormation stack so you can manage the resource's lifecycle with CloudFormation.

You can also use the import functionality if you want to move your resources between stacks, so you can better organize your stacks & resources.

### Drift Detection

Drift Detection gives you information on the difference between the current configuration of a resource and the configuration you declared in the template you used to create/update the resource.

Drift Detection only evaluate drift against properties that you explicitly declare in the template.

## Additional Topics

### Template Architecture

A template can be broken down to many smaller ones, each owned by a different team: vpc template owned by networking team, IAM template owned by security team and so on.

### Stack update behaviors

When you update a stack resource via template, AWS CloudFormation uses one of the following behaviors:
- Update with No Interruption: resource is updated without disrupting operation and changing its physical ID.
- Updates with Some Interruption: resource is updated with some interruption, but without changing its physical ID.
- Replacement: resource is recreated during an update with a new physical ID.

The method CloudFormation uses depends on which property gets updated for a given resource type, [reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html).

In case a resource will be recreated, you might want to have a backup strategy before performing the update.