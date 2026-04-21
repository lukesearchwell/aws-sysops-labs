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
 ├── Load-balancing-autoscaling-lab
 |   ├── architecture.md
 |   └── notes.md
 |
 ├── dataresilience-backup-lab
 |   ├── architecture.md
 |   └── notes.md
 |
 ├── ssm-lab
 │   └── notes.md
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
  - PowerShell
  - SSH
  </details>
</br>

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
   - PowerShell
  - SSH

</details>
</br>

### 3. Load Balancing and Autoscaling
> This lab demonstrates dynamic/elastic load balancing across multiple Availability Zones for delivery of high availability solutions.
<details>
<summary> Topics and Tools </summary>

#### Topics covered:
Application Load Balancer (ALB) creation
Target Group configuration and registration
Health check configuration and evaluation
Listener configuration and traffic forwarding rules
Multi-AZ subnet design for high availability
Security Group configuration for ALB and EC2 instances
Launch Template creation for EC2 configuration standardisation
Auto Scaling Group (ASG) deployment and configuration
Desired, minimum, and maximum capacity settings
Automatic instance registration with Target Groups
Instance health monitoring and replacement behaviour
Scaling policy creation based on CloudWatch metrics
CloudWatch CPU alarm configuration
Traffic distribution across multiple instances
Failure simulation and recovery validation
Infrastructure teardown and rebuild

#### Tools Used
- AWS CLI
- Amazon EC2
- Application Load Balancer (ALB)
- Target Groups
- Auto Scaling Groups (ASG)
- Launch Templates
- Amazon CloudWatch
- AWS IAM
- PowerShell
- SSH
</details>
</br>

### 4. Data Resilience and Backup
> This lab demonstrates dynamic/elastic load balancing across multiple Availability Zones for delivery of high availability solutions.
<details>
<summary> Topics and Tools </summary>

#### Topics covered:
RDS Multi-AZ deployment
DB subnet group creation
private database placement
security group configuration for EC2-to-RDS access
managed failover concepts
automated backups vs manual snapshots
EBS volume snapshot creation
snapshot-based volume restore
attaching restored storage to EC2
recovery workflow validation

#### Tools Used
- AWS CLI
- Amazon RDS
- Amazon EC2
- Amazon EBS
- EBS Snapshots
- DB Subnet Groups
- Security Groups
- Amazon CloudWatch
- AWS IAM
- PowerShell
- SSH
</details>

### 5. Systems Manager (SSM)
> This lab demonstrates implementation of AWS System Manager (SSM) which allows resources to be accessed via SSM, replacing SSH. SSM provides an agent-based approach to remote administration using IAM roles and outbound-only connectivity.
<details>
 <summary> Topics and Tools </summary>

  #### Topics covered:
  AWS SSM
IAM role-based access for EC2 management
SSM agent requirements and behaviour

  #### Tools Used
  - AWS CLI
  - Amazon EC2
  - AWS Systems Manager
  - AWS IAM
  - PowerShell
    </details>
</br>