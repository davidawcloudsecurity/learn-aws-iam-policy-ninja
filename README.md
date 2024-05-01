# learn-aws-iam-policy-ninja
How to secure aws resource with iam roles and policy

## Tools
Step 1: Know what policy to create

https://awspolicygen.s3.amazonaws.com/policygen.html

Step 2: Investigate action/policies/resources Can Be Access

https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html

Step 3: (Optional) This display live updates as you add. Conditions not working.

https://www.awsiamactions.io/generator

## Basic
Json-formatted documents, contain a statement that specifies:

1. What actions and by who (principal) can perform

2. Which resources can be accessed at which region by which account

3. Those actions, principals and resource can be access only if condition or conditions are met. (e.g AND or OR)

```ruby
{
  "Statement":[{
    "Effect":"<allow/deny>",
    "Principal":"<arn/iam/sts/role/user>",
    "Action":"<action>",
    "Resource":"<arn>",
    "Condition":{
      "<condition>":{
        "key":"<value>"
      }
    }
  ]
}
```
## Principal
Examples of an entity that is allowed or denied access to a resource
Principal here is referred to user/group or role attached
```ruby
<!-- Everyone (anyone with no reference) Similar to Power User but much powerful -->
"Principal":"AWS":"*.*"

<!-- Specific account or accounts -->
"Principal":{"AWS":"arn:aws:iam::123456789012:root"}
"Principal":{"AWS":"123456789012"}

<!-- Individual IAM user -->
"Principal":"AWS":"arn:aws:iam::123456789012:user/username"

<!-- Federated user (google/facebook) -->
"Principal":{"Federated":"www.amazon.com"}
"Principal":{"Federated":"graph.facebook.com"}
"Principal":{"Federated":"accounts.google.com"}

<!-- Specific role (Best Practise) -->
"Principal":{"AWS":"arn:aws:iam:123456789012:role/rolename"}

<!-- Specific service-->
"Principal":{"Service":"ec2.amazonaws.com"}
```
## Action
Describe the type of access that is allowed or denied

Statements must include either an _Action or NotAction_
```ruby
<!-- EC2 action -->
"Action":"ec2:StartInstances"

<!-- IAM action -->
"Action":"iam:ChangePassword"

<!-- Amazon S3 action -->
"Action":"s3:GetObject"

<!-- Specific multiple values for the Action element -->
"Action":["sqs:SendMessage", "sqs:ReceiveMessage"]
or
"Action":[
  "sqs:SendMessage",
  "sqs:ReceiveMessage"
]

<!-- wildcards (* or ?) in the action name. This covers create/delete/list/update -->
"Action":"iam:*AccessKey*"
```
## Understanding NotAction
It is not a deny but exclude and can be assume
```ruby
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "NotAllowToRunInstances",
      "NotAction": [
        "ec2:RunInstances"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```
## Understanding Deny
```ruby
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyToAllowToRunInstances",
      "Action": [
        "ec2:RunInstances"
      ],
      "Effect": "Deny",
      "Resource": "*"
    }
  ]
}
```
## Resource
```ruby
<-- S3 Bucket -->
"Resource":"arn:aws:s3:::my_corporate_bucket/*"

<-- Amazon SQS queue-->
"Resource":"arn:aws:sqs:us-west-2:123456789012:queue1"

<-- Multiple Amazon DynamoDB tables -->
"Resource":[
  "arn:aws:dynamodb:us-west-2:123456789012:table/books_table",
  "arn:aws:dynamodb:us-west-2:123456789012:table/magazines_table"
]

<-- All EC2 instances for an account in a region -->
"Resource":"arn:aws: ec2:us-east-1:123456789012:instance/*"
```
## Conditions
```ruby
{
"Version": "2012-10-17",
"Statement":[{
  "Sid": "AllowGroupToSeeBucketListInTheManagementConsole",
  "Action": [
    "s3:ListAllMyBuckets",
    "s3:GetBucketLocation"
  ], 
  "Effect": "Allow",
  "Resource": [
    "arn:aws:s3:::*"
  ]},
  {"Sid": "AllowRootLevelListingOfThisBucketAndHomePrefix",
  "Action": [
    "s3:ListBucket"
  ],
  "Effect": "Allow",
  "Resource": ["
    arn:aws:53:::myBucket"
  ],
  "Condition":{"StringEquals": {"s3:prefix":["", "Home/"],"s3:delimiter":["/"]}}},
  {"Sid": "AllowListBucketofASpecificUserPrefix",
  "Action": [
    "s3:ListBucket"
  ],
  "Effect": "Allow",
  "Resource": [
    "arn:aws:53:::myBucket"
  ],
  "Condition":{"StringLike":{"s3:prefix": ["Home/${aws:username}/*"]}}},
  {"Sid":"AllowUserFullAccess toJustSpecificUserPrefix",
  "Action": ["s3:*"],
  "Effect":"Allow",
  "Resource": [
    "arn:aws:53:::myBucket/Home/${aws:username]",
    "arn:aws:s3:::myBucket/Home/${aws:username}/*"
  ]}
}
```
```ruby
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "NotAction": ["iam: *","ec2: RunInstances"],
    "Resource":"*"},
  {
    "Effect": "Allow",
    "Action": "ec2: RunInstances",
    "NotResource":"arn:aws:ec2:*: 012345678912: instance/"},
  {
    "Effect": "Allow",
    "Action": "ec2: RunInstances",
    "Resource": [
        "arn: aws ec2:us-east-2:012345678912 :instance/", "arn:aws: ec2:eu-west-1: 012345678912: instance/*"],
    "Condition": {
        "StringLike": {"ec2: InstanceType": ["t1.*","t2.*", "m3.*"]}
    }
  ]
}
```

## StringNotLikeIFExist
```ruby
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "ec2:",
    "Resource": "*"
},
{
    "Effect": "Deny",
    "Action": "ec2: RunInstances"
    "Resource": "arn: aws ec2:*:012345678901:instance/*",
    "Condition": {
        "StringNotLikeIfExists": { "ec2: InstanceType": [
            "t1.*", "t2.*", "m3.*"
        ]
    }
  }
}
```
## Enforce Policy Workflow
```ruby
1. Decision starts at DENY
   1.1 What resource does the user require?
2. Evaluate all applicable policies
3. Is there an explicit DENY? No ==> Is there an ALLOW? No ==> Final decision=DENY
   ||                                        ||
   \/                                        \/
   Yes                                       YES
Final decision=DENY                      Final decision=ALLOW
- [ ] Need to test allow something but add deny all if this is workable.
```

## Troubleshooting
- [ ] https://youtu.be/y7-fAT3z8Lo?si=sONapM68KZ1ISl-p&t=3072

- [ ] decoding error message - https://youtu.be/y7-fAT3z8Lo?si=ioIGLqEY-jIEUiqM&t=3192

https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html

Policy Simulator - https://policysim.aws.amazon.com/home/index.jsp

## Tutorials
https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorials.html

## Hands On Labs

Create a home directory with s3 bucket using variable - https://youtu.be/y7-fAT3z8Lo?si=x3cZzTdVT2Q3zZTY&t=1251

Create a limited administrator - https://youtu.be/y7-fAT3z8Lo?si=BNyJE10vCVfbXs2B&t=1554

Grant conditional cross account access - https://youtu.be/y7-fAT3z8Lo?si=kujM7Mk2PzowT1BK&t=1969

https://catalog.us-east-1.prod.workshops.aws/workshops/8efd4edb-2b91-49fd-b1b8-3e3b5e71aa03/en-US/iam

https://catalog.us-east-1.prod.workshops.aws/workshops/f3a3e2bd-e1d5-49de-b8e6-dac361842e76/en-US/basic-modules/30-iam/iam

https://dev.to/aws-builders/hands-on-lab-introduction-to-iam-6ha

## Resource
https://blog.awsfundamentals.com/aws-iam-policies-a-practical-approach

https://aws.amazon.com/iam/resources/

https://blog.awsfundamentals.com/iam-policies-chatgpt-vs-github-copilot-vs-aws-policy-generator

## Best Practice
https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html

https://aws.amazon.com/iam/resources/best-practices/
