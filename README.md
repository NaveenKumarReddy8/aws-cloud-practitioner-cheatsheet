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
        "AWS": [
          "arn:aws:iam::xxxxxxxxxx:root"
        ]
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::mybucket/*"
      ]
    }
  ]
}
```

* **Version**: Policy language version.
* **Id [Optional]** An identifier for the policy.
* **Statement** One or more individual statements.
* **Sid [Optional]**: An identifier of the statement.
* **Effect [Allow, Deny]**: Whether statement allows or denies access.
* **Principal**: account/user/role to which this policy applied to.
* **Action**: list of actions this policy allows or denies.
* **Resources**: list of resources to which the actions applies to. **'*'** refers to all resources.

_Tags_ are optional and are present for almost all services in AWS. They are Key-Value pairs.

### Multi-Factor Authentication (MFA):

* Virtual MFA Device - Google Authenticator, Authy etc...
* Physical

### How to access AWS

* AWS Management Console: Password + MFA[Optional]
* AWS Command Line Interface (CLI): Access Keys
* AWS Software Development Kit (SDK)- For code: Access Keys

#### Using AWS CLI

After creating the access keys:

```shell
aws configure
```

and Enter the access key ID, secret access key, region name

### IAM Roles

IAM roles are for services to access the AWS.

Common roles:

* EC2 Instance roles
* Lambda function roles
* Roles for CloudFormation

### IAM Security Tools

* IAM Credentials Report (Account level): A report that lists all your account's users and status of their various
  credentials.
* IAM Access Advisor (user-level): Access Advisor shows the service permissions granted to a user and when those
  services were last accessed.

## Budget Setup

We can view the various bills generated by using the AWS services, look at free tier quota and predictions for breaching
free tier quota.

We can create budget which notifies whenever the budget is breached. Some templates are **Zero spend budget**, **Monthly
cost budget** etc...

## Elastic Compute Cloud (EC2)

* EC2 is one of the most popular of AWS's offerings.
* EC2 = Elastic Compute Cloud = Infrastructure as a service.
* It mainly consists in the capability of:
    * Renting Virtual machines (EC2)
    * Storing data on virtual drives (EBS)
    * Distributing load across machines (ELB)
    * Scaling the services using an auto-scaling group (ASG)

### EC2 Sizing & Configuration options

* Operating System (OS): Linux, Windows or macOS
* How much compute power & cores (CPU)
* How much RAM
* How much Storage space:
    * Network-attached (EBS & EFS)
    * Hardware (EC2 instance store)
* Network card: speed of the card, Public IP Address.
* Firewall rules: Security Group
* Bootstrap Script (configure at first launch): EC2 User Data

### EC2 User Data

* It is possible to bootstrap our instances using an **EC2 User Data** script.
* **bootstrapping** means launching commands when a machine starts.
* The script is only run once at the instance first start
* EC2 user data is used to automate boot tasks such as:
    * Installing updates
    * Installing software
    * Downloading common files from the internet etc...

#### Launching EC2 Instance

EC2 User Data:

```shell
#!/bin/bash
# Use this for your user data
# Install httpd (Linux 2 Version)
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```

#### EC2 Instance types - Overview

* You can use different types of EC2 instances that are optimised for different use cases.
    * General Purpose
        * Great for a diversity of workloads such as web servers or code repositories
        * Balance between
            * Compute
            * Memory
            * Networking
    * Compute Optimized
        * Great for Compute Intensive tasks that require high performance processors
            * Batch processing workloads
            * Media transcoding
            * High performance web servers
            * High performance computing (HPC)
            * Scientific modelling & machine learning
            * Dedicated gaming servers
    * Memory Optimized
        * Fast Performance for workloads that process large data in memory.
        * Use Cases:
            * High performance, relation/non-relational databases
            * Distributed web scale cache stores
            * In-memory databases optimized for BI(Business Intelligence)
            * Applications performing real-time processing of big structured data
    * Accelerated Computing
    * Storage Optimized
        * Great for storage-intensive tasks that require high, sequential read and write access to large data sets on
          local storage.
        * Use Cases:
            * High frequency online transaction processing (OLTP) systems
            * Relational & NoSQL databases
            * Cache for in-memory databases (ex: Redis)
            * Data warehousing applications
            * Distributed file systems
    * HPC Optimized
    * Instance Features
    * Measuring Instance Performance

* AWS has the following naming convention

`m5.2xlarge`

* **m**: Instance class
* **5**: Generation (AWS improves them over time)
* **2xlarge**: Size within the instance class

### Introduction to Security Groups

* Security groups are the fundamental of network security in AWS.
* They control how traffic is allowed into or out of our EC2 instances.
* Security groups only contain allow rules
* Security groups rules can reference by IP or by Security group.
* Security Groups are acting as a **firewall** on EC2 instances.
* They regulate:
    * Access to Ports
    * Authorised IP ranges - IPv4 and IPv6
    * Control of inbound network (from other to the instance)

Classic ports to know:

* 22: SSH(Secure Shell) - Log into a Linux instance
* 21: FTP (File Transfer Protocol) - Upload files into a file share
* 22: SFTP (Secure File Transfer Protocol) - Upload files using SSH
* 80: HTTP - Access Unsecured websites
* 443: HTTPS - Access Secured Websites
* 3389: RDP (Remote Desktop Protocol) - log into a windows instance.

**Security Group**
By default all outbound traffic is allowed.

**SSH into the EC2 instance created**

* Ensure that the security group is created which allows TCP on Port 22 for all IPs or your IP.
* Note the Public IP address of the instance.
* Default user created would be **ec2-user**
* You would require the PEM file

**_Note_**: You might require to chmod the pem file:

```shell
chmod 0400 ec2key.pem
```

```shell
ssh -i ec2key.pem ec2-user@54.84.219.178
```

**TIP**: Easier way to connect to EC2 instance is using **EC2 Instance connect**

#### EC2 Instance Purchasing Options:

* On-Demand Instances: Short workload, predictable pricing, pay by second
    * Linux os Windows: Billing per second, after the first minute
    * All other operating systems: Billing per hour
    * Hast the highest cost but no upfront payment
    * No long-term commitment
    * Recommended for short-term and un-interrupted workloads, where you can't predict how the application will behave.
* Reserved (1 & 3 years):
    * Reserved instances - long workloads - Recommended for steady-state usage applications (think Database)
        * You can buy and sell in the reserved instance Marketplace
    * Convertible Reserved instances: Long workloads with flexible instances
        * Can change teh EC2 instance type, instance family, OS, Scope and tenancy.
* Savings plans (1 & 3 years): Commitment to an amount of usage, long workload
    * Commit to a certain type of usage($10/hour for 1 or 3 years)
    * Usage beyond EC2 savings plan is billed at the On-Demand price
    * Locked to a specific instance family & AWS region (ex: M5 in us-east-1)
    * flexible across:
        * Instance size (ex: m5.xlarge, m5.2xlarge)
        * OS
        * Tenancy (Host, dedicated, Default)
* Spot instances: Short workloads, cheap, can lose instances (less reliable)
    * Can get upto 90% discount compared to on-demand
    * Instances that you can **lose** at any point of time if your max price is less than current spot price
    * The Most cost-efficient instances in AWS.
    * Useful for workloads that are resilient to failures:
        * Batch Jobs
        * Data Analysis
        * Image Processing
        * Any distributed workloads
        * Workloads with a flexible start and end time.
        * **Not suitable for critical jobs or databases**
* Dedicated Hosts: Book an entire physical server, control instance placement.
    * A physical server with EC2 instance capacity fully dedicated to your use.
    * Allows you to address **Compliance requirements** and use your existing server bound software licenses
    * Purchasing Options:
        * On-Demand: pay per second for active Dedicated host
        * Reserved: 1 or 3 years (No upfront, partial upfront, all upfront)
    * The most expensive option
    * Useful for software that have complicated licensing model (BYOL- Bring your own license)

#### Shared Responsibility Model for EC2:

* AWS:
    * Infrastructure (Global network security)
    * Isolation on Physical hosts
    * Replacing faulty hardware
    * Compliance Validation
* User:
    * Security Groups rules
    * Operating system patches and updates
    * Software and utilities installed on the EC2 instance
    * IAM Roles assigned to EC2 & IAM User access management
    * Data security on your instance