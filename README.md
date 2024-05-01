# learn-aws-iam-policy-ninja
How to secure aws resource with iam roles and policy

## Resources

https://awspolicygen.s3.amazonaws.com/policygen.html

## Basic
Json-formatted documents, contain a statement that specifies:

1. Which actions a principal can perform

2. Which resources can be accessed

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
  
}
```

## Tutorials & Hands On Labs

Tutorials - https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorials.html

Labs - https://catalog.us-east-1.prod.workshops.aws/workshops/8efd4edb-2b91-49fd-b1b8-3e3b5e71aa03/en-US/iam
