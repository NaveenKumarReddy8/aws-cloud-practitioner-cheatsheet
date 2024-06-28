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

## Elastic Block Volume (EBS)
* An EBS(Elastic Block Volume) is a network drive you can attach to your instances while they run
* It allows your instances to persist data, even after their termination
* They are bound to a specific availability zone.
* It can be detached from an EC2 instance and attached to another one quickly
* It uses network to communicate the instance, which means there might be a bit of latency.
* To move a volume across, you first need to snapshot it.

**Delete on Termination** controls the EBS behaviour when an EC2 instance terminates
* By default, the root EBS volume is deleted (attribute enabled)
* By default, any other attached EBS volume is not deleted (attribute disabled)


### EBS Snapshots
* Make a backup (snapshot) of your EBS volume at a point in time
* Not necessary to detach volume to do snapshot, but recommended
* Can copy snapshots across AZ or region

**EBS Snapshot Archive**

* Move a Snapshot to an **Archive Tier** that is 75% cheaper.
* Takes within 24 to 72 hours for restoring the archive.

**Recycle Bin for EBS Snapshots**

* Setup rules to retain deleted snapshots, so you can recover them after an accidental deletion.
* Specify retention (from 1 day to 1 year)

### Amazon Machine Image (AMI)
* AMI are a **customization** of an EC2 instance.
  * You add your own software, configuration, operating system, monitoring etc...
  * Faster boot / configuration time because all your software is pre-packaged.
* AMI are built for a **specific region** (and can be copied across regions)

We can launch EC2 Instances from:
* A public AMI: AWS Provided
* Your own AMI: You make and maintain them yourself
* An AWS Marketplace AMI: an AMI someone else made (and potentially sells)

### EC2 Image builder
* Used to automate the craetion of VM or container images.
* Automates the creation, maintain, validate and test EC2 AMIs
* Can be run on a schedule (weekly, whenever packages are updated, etc...)
* Free Service (only pay for the underlying resources)

### EC2 Instance Store
* EBS volumes are network drives with good but **Limited** performance.
* If you need a high performance hardware disk, use EC2 Instance Store.
* Better I/O performance
* EC2 Instance Store lose their storage if they're stopped (Ephemeral)
* Good for buffer/cache/scratch data/temporary content
* Risk of data loss if hardware fails
* Backups and replication are your responsibility
1. A company wants to rightsize its EC2 instances, which configuration change will meet this requirement will LEAST operational overhead?
A. Change the size and type of the EC2 instances based on utilization.
2. A company plans to host its data warehouse application on AWS. The company has a machine learning (ML) model and wants to use that model within its data warehouse for data forecasting. Which AWS service will meet these requirements?
A. Amazon Redshift ML
3. Which option is a pillar of thw AWS Wells architectured Framework?
   * Cost optimization
   * Operational excellence
   * Reliability
   * Performance efficiency
   * Security
   * Sustainability
4. A Company wants to migrate its application to the AWS Cloud. The company plans to identify and prioritize any business transformation oppurtunities and evaluate its AWS Cloud readiness . Which AWS service or tool should the company use to meet these requirements?
A. Cloud Adoption Framework (CAF)
5. A company is launching a critical business application in an AWS region, how can the company increase resilience for the application?
A. Deploy the application by using multiple AZs.
6. Which options are AWS CAF perspectives? (Select 2)
   * Business
   * People
   * Governance
   * Platform
   * Security
   * Operations
7. A company is defining its AWS multi-account strategy The company needs to control access to AWS Services and needs to consolidate billing across accounts. Which AWS service should the company use to meet these requirements?
A. AWS Organizations
8. Which options are common stakeholders for the AWS CAF platform perspective (Select 2)
   * CTO
   * Technology leaders
   * Architects
   * Engineers
9. Which AWS Service is deployed to VPCs and provide protection from common network threats?
A. AWS Network Firewall
10. A company has an environment that includes EC2 instances, Lightsail and on-prem servers. The company want so automate the security updates for its OS and applications. Which solution will meet these requirements with LEAST operational effort?
A. Use the AWS Systems manager Patch Manager capability
11. A company wants to query its server logs to gain insights about its customers experiences. Which AWS service will store this data **most cost efficiently**?
A. Amazon S3
12. A company is migration to AWS cloud and wants to optimize the use of its current software licenses. Which AWS services, features or purchasing options can teh company use to meet these requirements (Select 2)
A. Amazon EC2 Dedicated hosts, AWS License Manager
13. A company wants guidance to optimize the cost and performance of its current AWS environment which AWS service or tool should the company use to identify areas of for optimization?
A. AWS Trusted Advisor
14. Which AWS solution provides the ability for a company to run AWS services in the company's on-premises data center?
A. AWS Outposts
15. A company is using AWS Organizations to configure AWS accounts. Which design principle is a best practice for the company to implement?
A. Organize accounts based on security and operational needs
16. Which task can only an AWS account root user perform?
A. Changing the AWS Support plan
17. A company wants to use guidelines from the AWS Well architected framework to limit human error and facilitate responses to events. Which of the following is a well-architected design principle that will meet these requirements?
A. Perform Operations as code
18. A company encourages its teams to test failure scenarios regularly and to validate the understanding of impact of potential failures. Which pillar of AWS well architected framework does this philosophy represent?
A. Operational excellence
19. Which of the following are general AWS cloud design principles described in teh AWS well architected framework (Select 2)
    * Test systems at production scale
    * Drive architecture design based on data collected about the workload behaviour and requirements.
20. A company wants to use a managed security service for protection frm SQL injection attacks. The service also must provide detailed logging information about access to the company's ecommerce application. Which AWS service will meet these requirements?
A. AWS WAF
21. A company wants to automatically set up and govern a multi-account AWS environment. Which AWS Service provides the functionality?
A. AWS Control tower
22. A company wants to move its IOS application development and build activities to AWS. Which AWS Service or resources should the company use for these activities?
A. AWS Amplify
23. A company needs to apply security rules to a subnet for EC2 instances. Which AWS service or feature provide this functionality?
A. Network ACLs
24. A compnay is plannig to migrate to the AWs. The compnay wants to identify measurable business outcoes that will explain the value of the company's decision to migrate. Which phase of the cloud transformation journey it is?
A. Envision
25. A company migrated its core application onto multiple workloads in the AWs. The company wants to improve the applications reliability. Which cloud design principle should the company implement to achieve this goal?
A. Decouple the components
26. Which of the following is a benefit that AWS Professional services provides?
A. Advisory solutions for AWS adoption
27. What is the least expensive AWS support plan that provides the full set of AWS trusted advisor best practice checks for cost optimization?
A. AWS Business support


















