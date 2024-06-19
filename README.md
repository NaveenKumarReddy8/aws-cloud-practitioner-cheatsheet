# AWS Cloud Practitioner Cheat Sheet

## Shared Responsibility Model
* Customer = Responsibility for the security **in** the cloud.
* AWS = Responsibility for the security **of** the cloud.

## IAM - Identity and Access Management (Global Service)

Users or Groups can be assigned JSON documents called **Policies**

Ex:

```json
{
  "Version": "2012-10-17",
  "Id": "S3-account-permissions",
  "Statement": [
    {
      "Sid": "1",
      "Effect": "Allow",
      "Principal": {
        "AWS": ["arn:aws:iam::xxxxxxxxxx:root"]
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": ["arn:aws:s3:::mybucket/*"]
    }
  ]
}
```

* **Version**: Policy language version.
* **Id [Optional]** An identifier for the policy.
* **Statement** One or more individual statements.
* **Sid [Optional]**: An identifier of the statement.
* **Effect [Allow, Deny]**: Whether statement allows or denies access.