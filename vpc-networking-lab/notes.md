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
This EC2 instance will be hosted in the <b> public </b> subnet.</br>
Public address: 34.242.224.252 </br>
Private address: 10.0.1.139

#### CreateSecurityGroup
```
aws ec2 create-security-group 
    --group-name 'sysops-bastion-sg' 
    --description 'sysops-bastion-sg created 2026-03-16T11:23:46.702Z' 
    --vpc-id 'vpc-0457a6168fa286302' 
```
#### AuthorizeSecurityGroupIngress
```
aws ec2 authorize-security-group-ingress 
    --group-id 'sg-preview-1' 
    --ip-permissions '{"IpProtocol":"tcp","FromPort":22,"ToPort":22,"IpRanges":[{"CidrIp":"86.135.8.143/32"}]}' 
```
#### RunInstances
```
aws ec2 run-instances 
    --image-id 'ami-0d1b55a6d77a0c326' 
    --instance-type 't3.micro' 
    --key-name 'sysops-key-1' 
    --network-interfaces '{"SubnetId":"subnet-0a7688acd2ab74bc3","AssociatePublicIpAddress":true,"DeviceIndex":0,"Groups":["sg-preview-1"]}' 
    --credit-specification '{"CpuCredits":"unlimited"}' 
    --tag-specifications '{"ResourceType":"instance","Tags":[{"Key":"Name","Value":"sysops-bastion"}]}' 
    --metadata-options '{"HttpEndpoint":"enabled","HttpPutResponseHopLimit":2,"HttpTokens":"required"}' 
    --private-dns-name-options '{"HostnameType":"ip-name","EnableResourceNameDnsARecord":false,"EnableResourceNameDnsAAAARecord":false}' 
    --count '1' 
```
</br>

### EC2 App
This EC2 instance will be hosted in the <b> private</b> subnet. 
Private Address: 10.0.2.81

#### CreateSecurityGroup
```
aws ec2 create-security-group 
    --group-name 'sysops-app-sg' 
    --description 'sysops-app-sg created 2026-03-16T11:37:40.778Z' 
    --vpc-id 'vpc-0457a6168fa286302' 
```
#### AuthorizeSecurityGroupIngress
```
aws ec2 authorize-security-group-ingress 
    --group-id 'sg-preview-1' 
    --ip-permissions '{"IpProtocol":"tcp","FromPort":22,"ToPort":22,"UserIdGroupPairs":[{"GroupId":"sg-0905c371d80a3f7bb"}]}' 
```
#### RunInstances
```
aws ec2 run-instances 
    --image-id 'ami-0d1b55a6d77a0c326' 
    --instance-type 't3.micro' 
    --key-name 'sysops-key-1' 
    --network-interfaces '{"SubnetId":"subnet-022f5de6391c437fc","AssociatePublicIpAddress":false,"DeviceIndex":0,"Groups":["sg-preview-1"]}' 
    --credit-specification '{"CpuCredits":"unlimited"}' 
    --tag-specifications '{"ResourceType":"instance","Tags":[{"Key":"Name","Value":"sysops-app-instance"}]}' 
    --metadata-options '{"HttpEndpoint":"enabled","HttpPutResponseHopLimit":2,"HttpTokens":"required"}' 
    --private-dns-name-options '{"HostnameType":"ip-name","EnableResourceNameDnsARecord":false,"EnableResourceNameDnsAAAARecord":false}' 
    --count '1' 
```
## 6.0 Connectivity testing
I tested connectivity by using ssh to connect to the EC2 bastion and then used SSH forwarding to connect to the EC2 App instance.


SSH path verified:

Laptop → Bastion (public subnet) → Private EC2

Public instance reachable via Internet Gateway.
Private instance reachable only via bastion and VPC local routing.

### Troubleshooting
I encountered an issue with establishing connectivity between EC2 Bastion and the EC2 App Instance. The bastion host did not have access to the private key required to authenticate to the private instance, resulting in an SSH authentication failure.

To resolve, I re-established the SSH session from my local machine to the bastion host using <b>SSH agent forwarding</b>. <em>This allowed the bastion to temporarily use my local SSH credentials for authentication without requiring the private key to be copied or stored on the bastion host.</em>

Both EC2 instances were launched using the same key pair (sysops-key-1). By forwarding the SSH agent, the private key remained securely stored on my local machine while still allowing the bastion host to authenticate to the private instance.

#### Establish connectivity to Bastion (With agent forwarding)

```
ssh-add sysops-key-1.pem
ssh-A -i .../sysops-key-1.pem ec2-user@34.242.224.252
```
##### Verify SSH key forwarding

After connecting to the bastion host, I verified that the SSH agent had been forwarded from my local machine.
```
ssh-add -L
```

#### Establish connectivity to EC2 App Instance
I then established SSH connectivity to the EC2 App Instance.
```
ssh ec2-user@10.0.2.81
```
</br>

## 7.0 Teardown

All temporary resources were removed after testing.

Deleted:
- EC2 instances
- Subnets
- Route tables
- Internet Gateway
- Security groups

The `sysops-lab-vpc` was retained as the base network environment for future labs.