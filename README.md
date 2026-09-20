# AWS Cloud & Security Lab

A hands-on AWS laboratory focused on **cloud networking, security, IAM, Linux administration, network troubleshooting, and secure infrastructure design**.

The project was built to develop practical AWS skills and demonstrate how cloud infrastructure is designed, secured, tested, monitored, and documented.

---

## 🎯 Project Objectives

The main objectives of this lab are to:

* Build an AWS VPC network from scratch
* Understand VPC architecture and CIDR addressing
* Configure public and private subnets across multiple Availability Zones
* Deploy EC2 instances in different network zones
* Implement controlled SSH access using a bastion host
* Provide outbound Internet access to private instances through NAT Gateways
* Configure Security Groups and network access rules
* Learn IAM roles and policies
* Enable CloudTrail and VPC Flow Logs
* Analyze network traffic and troubleshoot connectivity
* Apply the principle of least privilege
* Document the infrastructure as a practical cloud/security portfolio project

---

# 🏗️ Architecture

The lab uses a custom VPC distributed across two Availability Zones.

```text
                                  AWS Cloud
                                      │
                              ┌───────▼────────┐
                              │      VPC       │
                              │  10.0.0.0/16    │
                              └───────┬────────┘
                                      │
                 ┌────────────────────┴────────────────────┐
                 │                                         │
              AZ-A                                       AZ-B
                 │                                         │
        ┌────────▼────────┐                       ┌────────▼────────┐
        │ Public Subnet A │                       │ Public Subnet B │
        │  10.0.1.0/24    │                       │  10.0.2.0/24    │
        └────────┬────────┘                       └────────┬────────┘
                 │                                         │
           Bastion EC2                              NAT Gateway B
                 │                                         │
                 │ SSH                                     │
                 ▼                                         │
        ┌─────────────────┐                       ┌─────────▼────────┐
        │ Private App A   │                       │                  │
        │ 10.0.11.0/24    │                       │                  │
        └────────┬────────┘                       │                  │
                 │                                │                  │
                 │                                │                  │
        ┌────────▼────────┐                       │                  │
        │ Private App B   │◄──────────────────────┘                  │
        │ 10.0.12.0/24    │                                          │
        └─────────────────┘                                          │
                                                                     │
                 Private instances use NAT for outbound Internet     │
                                                                     │
                         ┌───────────────────────────────────────────┘
                         │
                    Internet Gateway
                         │
                      Internet
```

> **Note:** The NAT Gateways were used during testing and were subsequently deleted to avoid ongoing AWS charges.

---

# 🌐 Network Design

## VPC

| Component                   | Configuration |
| --------------------------- | ------------- |
| Region                      | `us-east-1`   |
| VPC CIDR                    | `10.0.0.0/16` |
| Availability Zones          | 2             |
| Public subnets              | 2             |
| Private application subnets | 2             |
| NAT Gateways used           | 2             |
| Bastion host                | 1             |
| Private EC2 instances       | 2             |

VPC ID:

```text
vpc-06586aa30435a4d01
```

The VPC uses:

```text
10.0.0.0/16
```

which provides a private IPv4 address space that can be divided into multiple subnets.

---

## Public Subnets

### Public Subnet A

```text
10.0.1.0/24
```

Availability Zone:

```text
us-east-1a
```

### Public Subnet B

```text
10.0.2.0/24
```

Availability Zone:

```text
us-east-1b
```

A subnet is considered **public** when its associated route table provides a route to an Internet Gateway. Resources inside the subnet still require appropriate IP addressing and security rules to communicate with the Internet.

The bastion host was deployed in Public Subnet A.

---

## Private Application Subnets

### Private App Subnet A

```text
10.0.11.0/24
```

Availability Zone:

```text
us-east-1a
```

Example private EC2 address:

```text
10.0.11.151
```

### Private App Subnet B

```text
10.0.12.0/24
```

Availability Zone:

```text
us-east-1b
```

Example private EC2 address:

```text
10.0.12.53
```

The application instances were configured without public IP addresses.

This prevents them from being directly addressed from the public Internet.

---

# 🌍 Internet Gateway

An Internet Gateway (IGW) was attached to the VPC.

Public subnet routing follows the general architecture:

```text
EC2
 │
 ▼
Public Route Table
 │
 │ 0.0.0.0/0
 ▼
Internet Gateway
 │
 ▼
Internet
```

The Internet Gateway provides the VPC with a path to and from the Internet for resources that have appropriate public addressing and routing.

---

# 🔐 Bastion Host

A bastion host was used as a controlled administrative entry point into the private network.

Instead of exposing SSH directly on the private instances:

```text
Administrator
     │
     │ SSH
     ▼
Bastion
     │
     │ SSH
     ▼
Private EC2
```

The private EC2 instances do not have public IP addresses.

This design reduces their direct Internet exposure and centralizes administrative access through the bastion.

### Bastion details

Instance ID:

```text
i-01b7a01e858a30c44
```

Private IP:

```text
10.0.1.170
```

Security Group:

```text
SG-Bastion
```

---

# 🔒 Security Groups

Security Groups were used as the primary stateful network access control mechanism for the EC2 instances.

## Bastion Security Group

The bastion Security Group was used to control SSH access to the bastion host.

The exact source restriction should be documented in the accompanying screenshot/configuration evidence.

---

## Application Security Group

The private application instances use:

```text
SG-App
```

SSH access was restricted to traffic originating from the bastion Security Group rather than allowing SSH from the entire Internet.

Conceptually:

```text
SG-Bastion
     │
     │ TCP/22
     ▼
SG-App / Private EC2
```

The application Security Group also contained an HTTP rule for application/network testing.

---

# 🛣️ Route Tables

Separate routing was used for public and private subnets.

## Public Routing

The public subnets used a default route to the Internet Gateway:

```text
0.0.0.0/0
      │
      ▼
Internet Gateway
```

## Private Routing

The private subnets used NAT Gateways for outbound Internet connectivity.

During the active testing phase:

```text
Private EC2
    │
    ▼
Private Route Table
    │
    ▼
NAT Gateway
    │
    ▼
Internet Gateway
    │
    ▼
Internet
```

The private route tables were:

```text
private-rt-a
private-rt-b
```

`private-rt-a` was explicitly associated with the private application subnet in AZ-A.

---

# 🔄 NAT Gateways

Two NAT Gateways were used during the networking phase.

### NAT Gateway A

```text
NAT Gateway ID: nat-05adc1cef9c193a55
Private IP:     10.0.1.215
Public IP:      3.217.109.162
```

### NAT Gateway B

```text
NAT Gateway ID: nat-011f61a55c9f86895
Private IP:     10.0.2.88
Public IP:      34.228.114.194
```

Each private subnet used its corresponding NAT Gateway for outbound Internet access.

For example:

```text
Private Route Table A
0.0.0.0/0 → NAT Gateway A
```

and:

```text
Private Route Table B
0.0.0.0/0 → NAT Gateway B
```

This allowed the private EC2 instances to initiate outbound Internet connections without assigning public IP addresses to the instances.

### Cost management

After completing the networking tests, both NAT Gateways were deleted because NAT Gateways incur ongoing hourly charges.

The Elastic IP allocations associated with the NAT Gateways should also be checked and released if they are no longer required.

---

# 🖥️ EC2 Instances

## Bastion Host

Purpose:

* Administrative entry point
* SSH gateway to private instances
* Controlled access to the private network

Private IP:

```text
10.0.1.170
```

---

## Private Application A

Instance ID:

```text
i-0af77694f1cf3e5dc
```

Private IP:

```text
10.0.11.151
```

ENI:

```text
eni-0d1bb5ec20862566d
```

The instance had no public IP address.

---

## Private Application B

Instance ID:

```text
i-014ee0d850ee69b2f
```

Private IP:

```text
10.0.12.53
```

ENI:

```text
eni-0d1bef69b58049645
```

The instance had no public IP address.

---

# 🔑 SSH Access

The final administrative access path was:

```text
Administrator
     │
     │ SSH
     ▼
Bastion Host
     │
     │ SSH
     ▼
Private EC2
```

SSH access to the private EC2 instances was not exposed directly to the Internet.

Linux file permissions were also used to protect the private SSH key.

---

# 🧪 Connectivity Testing

Several tests were performed to validate the infrastructure.

## Private EC2 → Internet

From a private EC2 instance:

```bash
curl -I https://aws.amazon.com
```

The request successfully returned an HTTP 200 response.

This demonstrated that the private instance could reach the Internet through the NAT Gateway while the instance itself had no public IP address.

---

## Public Egress IP

The following command was used:

```bash
curl -s https://checkip.amazonaws.com
```

The private instances returned the public IP addresses associated with their NAT Gateways.

App A:

```text
3.217.109.162
```

App B:

```text
34.228.114.194
```

This demonstrated the path:

```text
Private EC2
    ↓
NAT Gateway
    ↓
Public NAT IP
    ↓
Internet
```

---

## Routing Verification

The routing table was inspected using:

```bash
ip route
```

App A showed a default route through:

```text
10.0.11.1
```

App B showed a default route through:

```text
10.0.12.1
```

These are the subnet-level VPC router addresses used by the instances.

---

## Private EC2 → Private EC2

Connectivity between the private instances was tested using ICMP:

```bash
ping 10.0.11.151
```

The test initially failed, after which the relevant network/security configuration was investigated.

The final configuration allowed private-to-private connectivity successfully.

This demonstrated that instances in different private subnets could communicate through the VPC's internal routing.

---

# 📊 VPC Flow Logs

VPC Flow Logs were configured to provide visibility into network traffic.

Configuration:

```text
Flow Log ID:
fl-04b59f0eb76c7f513

Traffic type:
All

Destination:
S3

Region:
us-east-1

Encryption:
SSE-S3
```

The logs were stored in:

```text
vpc-flow-logs-458575205409-ayb
```

under:

```text
AWSLogs/458575205409/vpcflowlogs/us-east-1/
```

Flow Logs can be used for:

* Network troubleshooting
* Investigating rejected traffic
* Security investigations
* Identifying unusual traffic patterns
* Understanding network behavior
* Auditing network flows

---

## Flow Log Analysis

A downloaded Flow Log contained records such as:

```text
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
```

Several records were associated with the NAT Gateway ENIs:

```text
eni-007e9247c3198bc26
```

and:

```text
eni-09faf44014196404b
```

These corresponded to the NAT Gateway network interfaces.

Observed traffic included connections to ports such as:

```text
23
2375
3259
5900
6379
```

from various public Internet addresses.

### Important interpretation

These records **should not be interpreted as proof that the private EC2 instances were attacked or compromised**.

In particular:

```text
ACCEPT
```

in a VPC Flow Log means that the traffic was accepted at the network/flow level. It does not by itself prove that:

* an application accepted the connection
* a service was listening
* the connection reached a particular application
* authentication succeeded
* an attacker compromised a system

The observed records were primarily associated with NAT Gateway ENIs, which had public Internet-facing addresses.

The traffic pattern was consistent with automated Internet scanning/probing that commonly occurs against public IP addresses.

The next investigation step is to verify the Flow Log's **resource type** and determine whether records for the private EC2 ENIs are present:

```text
App A:
eni-0d1bb5ec20862566d

App B:
eni-0d1bef69b58049645
```

This distinction is important when analyzing whether traffic was observed at the NAT Gateway or directly at the application instances.

---

# ☁️ CloudTrail

AWS CloudTrail was enabled to record AWS API activity.

Trail:

```text
management-events
```

Region:

```text
us-east-1
```

S3 destination:

```text
aws-cloudtrail-logs-458575205409-1400ea23
```

CloudTrail logs were observed in the S3 bucket under:

```text
AWSLogs/458575205409/CloudTrail/us-east-1/
```

CloudTrail provides an audit trail of AWS API activity and is useful for investigating changes to AWS resources and identifying which identities performed actions.

---

# 🔐 IAM

IAM was studied as part of the security portion of the lab.

The lab covers:

* IAM users
* IAM groups
* IAM roles
* IAM policies
* Trust policies
* Permission policies
* Managed policies
* Inline policies
* Least privilege

The practical IAM role exercise is still being developed and will be documented separately once the role, trust relationship, permissions, and permission tests are completed.

---

# 📜 IAM Policies

An IAM policy defines permissions using elements such as:

```text
Effect
Action
Resource
Condition
```

Example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

This example allows an identity to retrieve objects from the specified S3 bucket.

---

# 🛡️ Principle of Least Privilege

The lab follows the principle of:

> Give identities only the permissions they actually need.

For example, an application that only needs to read objects from S3 should not automatically receive:

```text
s3:*
```

when it only requires:

```text
s3:GetObject
```

Least privilege reduces the potential impact of compromised credentials or compromised workloads.

---

# 👤 IAM Roles

IAM roles allow AWS services and workloads to obtain temporary credentials without storing long-term access keys inside servers.

Conceptually:

```text
EC2
 │
 ▼
IAM Role
 │
 ▼
Permission Policy
 │
 ▼
AWS Resource
```

This is preferable to storing long-term credentials such as:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

inside an EC2 instance.

---

# 🤝 Trust Policy vs Permission Policy

One of the key IAM concepts studied in this project is the difference between trust and permissions.

### Trust Policy

Defines:

> Who or what is allowed to assume the role?

For example:

```text
EC2 service
     │
     ▼
Trust Policy
     │
     ▼
IAM Role
```

### Permission Policy

Defines:

> What can the role do after it has been assumed?

For example:

```text
IAM Role
     │
     ▼
Permission Policy
     │
     ▼
s3:GetObject
```

In short:

```text
Trust Policy
= WHO can assume the role

Permission Policy
= WHAT the role can do
```

---

# 🚫 Explicit Deny

IAM authorization follows an important rule:

```text
Explicit Deny
      ↓
Overrides Allow
```

If an applicable policy explicitly denies an action, the explicit Deny takes precedence over an Allow.

If there is no applicable Allow, the request is denied by default.

This results in a secure default model:

```text
No permission
     ↓
Denied
```

---

# 🧰 Technologies Used

## AWS

* Amazon VPC
* Amazon EC2
* Internet Gateway
* NAT Gateway
* Route Tables
* Security Groups
* VPC Flow Logs
* IAM
* AWS CloudTrail
* Amazon S3
* Availability Zones

## Networking

* IPv4
* CIDR
* Subnets
* Routing
* NAT
* Internet Gateway
* Public/private networking
* Security Groups
* Network traffic monitoring

## Linux

* SSH
* `curl`
* `ping`
* `ip route`
* Linux file permissions
* Network troubleshooting

---

# 🧪 Troubleshooting Performed

The project involved troubleshooting several real-world infrastructure problems.

### SSH Connectivity

SSH access to the bastion host was configured and tested.

Private EC2 SSH connectivity was then established through the bastion.

---

### Private Internet Connectivity

Private EC2 instances required correct:

* Route table associations
* NAT Gateway configuration
* Security Group rules
* Subnet configuration

Connectivity was verified with:

```bash
curl -I https://aws.amazon.com
```

and:

```bash
curl -s https://checkip.amazonaws.com
```

---

### Private Instance Connectivity

Connectivity between the private EC2 instances was tested.

Initial connectivity issues were investigated and the relevant network/security configuration was corrected.

---

### VPC Flow Log Analysis

Flow Log records were downloaded from S3 and analyzed.

The analysis demonstrated the importance of identifying the specific ENI associated with a flow before interpreting network traffic.

---

### AWS Resource Cost Management

After completing the NAT networking tests, both NAT Gateways were deleted to prevent unnecessary ongoing charges.

EC2 instances were also stopped after testing.

---

# 📸 Recommended GitHub Evidence

Screenshots should be included in the repository to demonstrate the actual implementation.

Recommended evidence:

```text
screenshots/
├── vpc.png
├── subnets.png
├── route-tables.png
├── nat-gateways.png
├── internet-gateway.png
├── ec2-instances.png
├── bastion-ssh.png
├── private-ec2.png
├── private-connectivity.png
├── nat-internet-test.png
├── security-groups.png
├── cloudtrail.png
├── iam-role.png
├── iam-policy.png
└── vpc-flow-logs.png
```

Do not upload screenshots containing:

* AWS access keys
* Secret keys
* Passwords
* Private SSH keys
* Sensitive credentials
* Unnecessary account information

---

# 📁 Repository Structure

```text
aws-cloud-security-lab/
│
├── README.md
│
├── 01-aws-fundamentals/
│   └── README.md
│
├── 02-networking/
│   ├── README.md
│   ├── architecture/
│   ├── screenshots/
│   └── tests/
│
├── 03-iam/
│   ├── README.md
│   ├── policies/
│   └── screenshots/
│
├── 04-cloud-security/
│   ├── README.md
│   ├── flow-logs/
│   └── screenshots/
│
├── 05-terraform/
│   └── README.md
│
└── 06-projects/
    └── README.md
```

---

# 📈 Skills Demonstrated

This project demonstrates practical exposure to:

* AWS networking
* VPC architecture
* Subnet design
* CIDR addressing
* Route tables
* Internet Gateways
* NAT Gateways
* EC2
* Linux administration
* SSH
* Bastion architecture
* Security Groups
* IAM policies
* IAM roles
* IAM trust policies
* Least privilege
* CloudTrail
* VPC Flow Logs
* Network troubleshooting
* Cloud security fundamentals
* AWS cost awareness

---

# 💼 Career Relevance

The project is designed to demonstrate practical skills relevant to entry-level roles such as:

* Cloud Support Engineer
* Junior Cloud Engineer
* Network Administrator
* Network Security Technician
* Cloud Security Junior
* Infrastructure Support Engineer
* Junior DevSecOps Engineer

The goal is not simply to demonstrate that AWS services can be created through the AWS Console.

The project focuses on understanding:

**why the components exist, how they communicate, how access is controlled, how traffic is monitored, and how infrastructure problems are diagnosed.**

---

# 🚀 Future Improvements

Planned extensions include:

## Infrastructure as Code

Rebuild the architecture using:

* Terraform
* AWS CLI

## Monitoring

Add:

* Amazon CloudWatch
* CloudWatch alarms
* Centralized logging

## Security

Add:

* AWS Config
* Amazon GuardDuty
* AWS Security Hub

## Application Layer

Deploy an actual application behind:

```text
Internet
    ↓
Application Load Balancer
    ↓
Private EC2
```

## High Availability

Expand the architecture to improve redundancy across multiple Availability Zones.

## Automation

Introduce:

* CI/CD
* GitHub Actions
* Terraform automation
* Automated security checks

---

# 📚 Key Lessons Learned

1. **A subnet is public or private primarily because of its routing configuration, not its name.**

2. **A private EC2 instance can access the Internet without having a public IP by using a NAT Gateway.**

3. **Security Groups control network access to AWS resources and are stateful.**

4. **A bastion host can provide a controlled administrative path to private servers.**

5. **IAM trust policies determine who or what can assume a role, while permission policies determine what the role can do.**

6. **Least privilege reduces unnecessary permissions and limits the potential impact of compromised identities.**

7. **VPC Flow Logs provide useful network visibility, but individual flow records must be interpreted in the context of the associated network interface.**

8. **An ****`ACCEPT`**** flow-log record does not by itself prove that an application accepted a connection or that a system was compromised.**

9. **Cloud networking problems should be diagnosed systematically by checking IP addressing, route tables, Security Groups, interfaces, and connectivity tests.**

10. **Cloud security also includes cost awareness: temporary resources such as NAT Gateways should be removed when they are no longer required.**

---

# 🚧 Project Status

**In Progress**

Completed components include:

* [x] VPC
* [x] Two Availability Zones
* [x] Two public subnets
* [x] Two private application subnets
* [x] Internet Gateway
* [x] Route tables
* [x] Bastion host
* [x] Two private EC2 instances
* [x] Security Groups
* [x] NAT Gateway networking tests
* [x] Private-to-private connectivity testing
* [x] Private-to-Internet connectivity testing
* [x] CloudTrail
* [x] VPC Flow Logs
* [x] Flow Log investigation
* [ ] Complete practical IAM role exercise
* [x] Verify Flow Log resource scope and App ENI records
* [ ] Terraform implementation
* [ ] Advanced cloud-security project

NAT Gateways used during testing have been deleted to avoid ongoing charges, and the EC2 instances were stopped after testing.

---

##  Author

**Ayoub Boudouh**

Network Administration & Security

Focused on:

**Cloud → Cloud Security → Network Security → DevSecOps**
# aws-cloud-portfolio
