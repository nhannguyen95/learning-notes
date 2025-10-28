|**Resource**     |**Bounding level**                     |
|-----------------|---------------------------------------|
|NACL             |Subnet                                 |
|VM               |Subnet                                 |
|Subnet           |AZ                                     |
|ENI              |AZ                                     |
|EBS              |AZ                                     |
|Security Group   |VPC (with exceptions)                  |
|S3               |Region (but operate at global level)   |
|IGW              |Region (can only be attached to 1 VPC) |
|RDS              |Region                                 |
|DynamoDB         |Region                                 |
|EFS              |Region                                 |
|SQS              |Region                                 |
|ECS              |Region                                 |
|Endpoint         |Region                                 |
|VPC              |Region                                 |
|Elastic IP       |Region (but operate at global level)   |
|IAM              |Global                                 |
|CloudFront       |Global                                 |
|Route53          |Global                                 |
