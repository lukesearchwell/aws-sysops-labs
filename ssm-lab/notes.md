# Load Balancing and Autoscaling
## Objective: 
This lab focuses on replacing traditional SSH-based access with AWS Systems Manager. An EC2 instance was configured with the required IAM role and SSM agent, allowing remote access via Session Manager and command execution via Run Command. SSH access (port 22) was removed to validate that instance management could be performed entirely through SSM.


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

## 1.0 Launch EC2 instance
Launched an EC2 instance via AWS console within public subnet. Created new security group and placed the EC2 in a private subnet.
```shell
aws ec2 run-instances 
--image-id 'ami-0442403fb8d244144' 
--instance-type 't3.micro' 
--network-interfaces '{"SubnetId":"subnet-04cc5a8c5bd26e94a","AssociatePublicIpAddress":false,"DeviceIndex":0,"Groups":["sg-06041754a5c1460d6"]}' 
--credit-specification '{"CpuCredits":"unlimited"}' 
--tag-specifications '{"ResourceType":"instance","Tags":[{"Key":"Name","Value":"ec2-ssm-lab"}]}' 
--metadata-options '{"HttpEndpoint":"enabled","HttpPutResponseHopLimit":2,"HttpTokens":"required"}' 
--private-dns-name-options '{"HostnameType":"ip-name","EnableResourceNameDnsARecord":false,"EnableResourceNameDnsAAAARecord":false}' 
--count '1' 
```

## 2.0 IAM Role
Create the IAM role ``sysops-ssm-role`` that will be used to enable SSM.
```shell
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Principal": {
                "Service": [
                    "ec2.amazonaws.com"
                ]
            }
        }
    ]
}
```


##Troubleshooting
I had to add a NAT Gateway so that the instance could reach the SSM service. For the purpose of this lab, I created new, dedicated infrasture:
- Security group
- Subnet
- NAT Gateway


## Observations
Accessing the EC2 instance via SSM allows there to be a dedicated control plane for instance administration without requiring network access. 

I did not need to keep a key locally, I did not need to set up multiple hosts with multiple different routes - this approach allowed me to access the CLI of the service much more efficiently than using SSH from my machine.

Key observation is that upon using `ifconfig` the EC2 instance had the private address of 10.0.30.9. No public ip address needed to be assigned to the instance.

## Teardown
All created infrastructure was removed upon conclusion of this lab.