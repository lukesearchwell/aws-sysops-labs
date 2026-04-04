# Load Balancing and Autoscaling
## Objective: 
Design and deploy a high availability and scalable  architecture on AWS using an Application Load Balancer (ALB), Target Group, and Auto Scaling Group (ASG). I will also send logs to the CloudWatch CPU Alarm to enable the Auto Scaling group to operate as intended.

This lab demonstrates how to traffic can be distributed across multiple instances and availability zones, monitor health, and dynamically adjust compute capacity based on demand.


### Key concepts demonstrated:
- Custom VPC creation
- Application Load Balancer (ALB) deployment across multiple Availability Zones
- Target Group configuration and health check evaluation
- Traffic distribution across multiple EC2 instances
- Launch Template creation for standardised instance configuration
- Auto Scaling Group (ASG) setup for dynamic capacity management
- Desired, minimum, and maximum capacity configuration
- Automatic registration and deregistration of instances in Target Groups
- CloudWatch CPU alarm configuration for scaling triggers
- Scaling policies based on performance metrics
- High availability through multi-AZ architecture
- Separation of public and private subnets for secure design
- Failure handling and instance replacement via ASG


### Environment
```
Region: eu-west-1
VPC: sysops-lab-vpc


```

## 1.0 Prerequisites/setup

### Security Groups
Security Groups created via AWS CLI. The ALB security group is configured to allow all inbound HTTP traffic over port 80 and all outbound traffic. The web app security group is configured to allow inbound HTTP traffic <b> only from 'sysops-alb-sg'.</b> I also added another rule to allow SSH from my IP address or from a bastion host so that I can directly connect to any instances within the security group.

Both security groups allow all outbound traffic.

#### Application Load Balancer (ALB) Security Group

```shell
aws ec2 'create-security-group' 
    --group-name 'sysops-alb-sg' 
    --description 'Application load balancer sg' 
    --vpc-id 'vpc-0457a6168fa286302' 
```
#### Web Application Security Group
```shell
aws ec2 'create-security-group' 
    --group-name 'sysops-web-sg' 
    --description 'Web application sg' 
    --vpc-id 'vpc-0457a6168fa286302' 
```

### Subnets

| Name | Subnet ID | CIDR | Route Table |
| --- | --- | --- | --- | --- |
| sysops-public-az1 | subnet-08c1815cc117097ab | 10.0.1.0/24 | sysops-public-rt |
| sysops-public-az2 | subnet-0b5ca290d0e6f4d53 | 10.0.2.0/24 | sysops-public-rt |
| sysops-private-az1 | subnet-04cc5a8c5bd26e94a | 10.0.10.0/24 | sysops-private-rt |
| sysops-private-az2 | subnet-0a1be5fb7193fb676 | 10.0.20.0/24 | sysops-private-rt |

### EC2 Test
I set up a new E2 instance within 'sysops-web-sg'

## 2.0 Target Group

## 3.0 Application Load Balancer

## 4.0 Create EC2 launch template

## 5.0 Auto Scaling Group
### 5.1 Scaling Policy
### 5.2 CloudWatch CPU alarm