# AWS SysOps Labs

### Repository Structure

``` 
  aws-sysops-labs
 │
 ├── ec2-iam-lab
 │   └── notes.md
 |
 ├── VPC-networking-lab
 |   ├── architecture.md
 |   └── notes.md
 |
 └── README.md
```

## Content Overview

### 1. EC2 + IAM Roles
> This lab demonstrates secure  access from an EC2 instance and implementation of IAM roles.
<details>
  <summary> Topics and Tools </summary>

  #### Topics covered:
  Launching EC2 instance via AWS Console
  Launching EC2 instance via AWS CLI
  Amazon Machine Image (AMI) discovery
  IAM role creation and attachment
  Instance Metadata Service (IMDSv2)
  Retrieval of temporary credentials
  S3 access using role-based authentication
  IAM permission testing 
  Infrastructure teardown

  #### Tools Used
  - AWS CLI
  - Amazon EC2
  - AWS IAM
  - Amazon S3
  - Git
  - PowerShell
  - SSH
  </details>

### 2. VPC Networking
> This lab demonstrates cloud networking and creation of a VPC including specification of inbound and outbound rules.
<details>
  <summary> Topics and Tools </summary>

  #### Topics covered:
  Custom VPC creation
  CIDR block planning and subnet segmentation
  Public vs private subnet architecture
  Internet Gateway configuration
  Route table creation and subnet association
  Default route (0.0.0.0/0) configuration
  Network connectivity troubleshooting
  Route table misconfiguration debugging
  Security Groups vs Network ACL behavioural differences
  Stateful vs stateless firewall concepts
  Inbound vs outbound traffic filtering
  VPC Flow Logs configuration and analysis
  Network traffic observation (ACCEPT vs REJECT)
  Infrastructure teardown

  #### Tools Used
  - AWS CLI
  - Amazon VPC
  - Amazon EC2
  - Internet Gateway
  - Route Tables
  - Security Groups
  - Network ACLs
  - VPC Flow Logs
  - Amazon CloudWatch Logs
  - Git
  - PowerShell
  - SSH

</details>
