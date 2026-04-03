# Load Balancing and Autoscaling
## Objective: 
Design and deploy a high availability and scalable  architecture on AWS using an Application Load Balancer (ALB), Target Group, and Auto Scaling Group (ASG). 

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

## 1.0 Build Custom VPC
