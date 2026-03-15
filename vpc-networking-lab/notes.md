# VPC Networking Lab
## Objective: 
Demonstrate secure access to AWS services from an EC2 instance using an IAM role and temporary credentials, avoiding the use of long-lived access keys on the instance.

### Key concepts demonstrated:
- Custom VPC creation
- Public vs private subnet architecture
- Network segmentation
- Internet Gateway configuration
- Route table creation and subnet association
- Default route (0.0.0.0/0) configuration
- Network connectivity troubleshooting
- Route table misconfiguration debugging
- Security Groups vs Network ACL behavioural differences
- Stateful vs stateless firewall concepts
- Inbound vs outbound traffic filtering
- VPC Flow Logs configuration and analysis
- Network traffic observation (ACCEPT vs REJECT)

### Environment
```
Region: eu-west-1
Instance type used: t3.micro
Operating system: Amazon Linux 2023
```

## 1.0 Build Custom VPC
VPC created via AWS console
```
Name: sysops-lab-vpc
CIDR: 10.0.0.0/16
VPC ID: vpc-0457a6168fa286302
```

## 2.0 Subnets
I created 2 subnets. </br>
| Name | Address | subnet ID |
| --- | --- | ---|
| public subnet | 10.0.1.0/24 | subnet-0a7688acd2ab74bc3 |
| private subnet | 10.0.2.0/24 | subnet-022f5de6391c437fc |

Subnets created within AWS CLI. I tried to break the script over multiple lines for better readability but bash would not recognise that the lines were part of the same script.  


### Formatting in Bash...
This script format failed:
```
aws ec2 create-subnet`
>--vpc-id vpc-0457a6168fa286302'
>--cidr-block 10.0.x.0/24
```

This script format was successful:
```
aws ec2 create-subnet --vpc-id pc-0457a6168fa286302 --cidr-block 10.0.x.0.24
```

In bash, I should use "\" to write scripts over multiple lines.
```
aws ec2 create-subnet\
 --vpi-id pc-0457a6168fa286302
 --cidr-block 10.0.x.0.24
```


## 3.0 Internet Gateway
Gateway created and attached to sysops-lab-vpc via AWS Console.
```
Name: sysops-lab-igw
```

## 4.0 Route Tables
```
Public: sysops-public-rt
Private: sysops-private-rt 
```
I created public and private route tables using the AWS console. I assigned both tables their local route which provides a path to the VPC subnet (10.0.0.0/16).

The public route table was also assigned a 0.0.0.0/0 which allows traffic to <b> all IPV4 destinations. </b> The target was specified as sysops-lab-igw which will allow internet traffic ingress and egress by resources within the public subnet, using sysops-public-rt. 

This route alone does not make a resource  internet-accessible; the instance must also have a public IP address (Elastic IP) and be confirgured with permissive security controls. Access will be further governed through Security Groups/Network ACLs.

## 5.0 EC2 Instances
I created two EC2 instanced for this lab. 

### EC2 Bastion
This EC2 instance will be hosted in the <b> public </b> subnet.
```
name: sysops-ec2-bastion
address:
```

</br>

### EC2 App
This EC2 instance will be hosted in the <b> private</b> subnet
```
name: sysops-ec2-app
address:
```