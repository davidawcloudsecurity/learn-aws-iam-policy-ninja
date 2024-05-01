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
