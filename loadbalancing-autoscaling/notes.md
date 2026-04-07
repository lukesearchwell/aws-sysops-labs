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
| --- | --- | --- | --- | 
| sysops-public-az1 | subnet-08c1815cc117097ab | 10.0.1.0/24 | sysops-public-rt |
| sysops-public-az2 | subnet-0b5ca290d0e6f4d53 | 10.0.2.0/24 | sysops-public-rt |
| sysops-private-az1 | subnet-04cc5a8c5bd26e94a | 10.0.10.0/24 | sysops-private-rt |
| sysops-private-az2 | subnet-0a1be5fb7193fb676 | 10.0.20.0/24 | sysops-private-rt |

### EC2 Test
I set up a new EC2 instance within 'sysops-web-sg' called "sysops-web-1". This instance was created manually and secured with a new RSA key pair.

#### Web app EC2 config
```shell
aws ec2 run-instances --image-id 'ami-0762bad84218d1ffa' --instance-type 't3.micro' --key-name '20260404-sysops-key' --network-interfaces '{"SubnetId":"subnet-04cc5a8c5bd26e94a","AssociatePublicIpAddress":false,"DeviceIndex":0,"Groups":["sg-0c8f4c871aab02969"]}' --credit-specification '{"CpuCredits":"unlimited"}' --tag-specifications '{"ResourceType":"instance","Tags":[{"Key":"Name","Value":"sysops-web-1"}]}' --metadata-options '{"HttpEndpoint":"enabled","HttpPutResponseHopLimit":2,"HttpTokens":"required"}' --private-dns-name-options '{"HostnameType":"ip-name","EnableResourceNameDnsARecord":false,"EnableResourceNameDnsAAAARecord":false}' --count '1' 
```
Instance is in the private subnet, to test connectivity via the security group, I created a bastion host "sysops-alb-bastion" within the public subnet using the ALB security group. This allowed me to SSH in and then transfer the key to access sysops-web-1 via SSH. I had to amend the security group rules as I initially added SSH access from my IP address to sysops-web-sg.

This was the incorrect configuration as no instances using the security group should have a public ip address. I had to add a new inbound traffic rule to allow SSH traffic over port 22 from sysops-alb-sg to allow me to access instances via a bastion host. 

#### Bastion host EC2 Config
```shell
aws ec2 run-instances --image-id 'ami-0762bad84218d1ffa' --instance-type 't3.micro' --key-name '20260404-sysops-key' --network-interfaces '{"SubnetId":"subnet-08c1815cc117097ab","AssociatePublicIpAddress":true,"DeviceIndex":0,"Groups":["sg-0d9290a1fed82394d"]}' --credit-specification '{"CpuCredits":"unlimited"}' --tag-specifications '{"ResourceType":"instance","Tags":[{"Key":"Name","Value":"sysops-alb-bastion"}]}' --metadata-options '{"HttpEndpoint":"enabled","HttpPutResponseHopLimit":2,"HttpTokens":"required"}' --private-dns-name-options '{"HostnameType":"ip-name","EnableResourceNameDnsARecord":false,"EnableResourceNameDnsAAAARecord":false}' --count '1' 
```

#### Web server installation
I used the following to install nginx. 

```bash
sudo dnf install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
echo "Hello from $(hostname)" | sudo tee /usr/share/nginx/html/index.html
curl localhost
```
Upon correct configuration the echo returns a string with hostname details.

As the web app did not have a connection to the internet, I installed nginx to the bastion host and deleted the web app EC2 instance. I reconfigured the Bastion host after installing nginx to be 'sysops-web-bastion' and assigned it to 'sysops-web-sg'. 

## 2.0 Target Group
I used the console to create the Target Group.

## 3.0 Application Load Balancer
```
Name: sysops-alb 
Scheme: Internet-facing 
IP address type: IPv4
```

VPC: 
```
vpc-0457a6168fa286302
```

Availability Zones and subnets: 
``` 
eu-west-1a 
subnet-0b5ca290d0e6f4d53 
sysops-public-az2 

eu-west-1c 
subnet-08c1815cc117097ab
sysops-public-az1
```
Security groups: 
```
sysops-alb-sg 
sg-0d9290a1fed82394d 
```
Listeners and routing 
```
HTTP:80 | Forward to 1 target group
```
## 4.0 Create EC2 launch template
Created 'sysops-web-lt' as EC2 launch template. This will standardise the configuration of any EC2 instances that are launched to be web servers via the Auto Scaling Group (ASG).

Added the following to User data to ensure that nginx is installed upon first launch of each instance so that all instances become web servers automatically.
```bash
#!/bin/bash
dnf install -y nginx
systemctl enable nginx
systemctl start nginx
echo "Hello from $(hostname)" > /usr/share/nginx/html/index.html
```

## 5.0 Auto Scaling Group
Created ASG 'sysops-web-asg' to trigger launch of EC2 instances via launch template.
### 5.1 Scaling Policy
Instance scaling to occur based on CPU load. At 50-60% average CPU usage, a new instance should be launched

Cloudwatch was configured within the ASG settings, meaning separate configuration activity directly in Cloudwatch was unneeded.

SNS Alerting was also configured here. Notifications successfully received for instance state changes at the registered email address.

## 6.0 Testing
### 6.1 CPU Load testing 
To test the auto scaling capability, I used the following script on a live instance.
```bash
yes > /dev/null &
```
This caused the instance to output "y" continuously, as fast as possible which will emulate high CPU usage on an individual instance. Once CPU usage reaches the limit specified in the scaling policy, new instances were created.

I then terminated the process by using:
```bash
pkill yes
```
Both processes worked as intended. This justified that the Auto-scaling group was working as intended based on the CPU usage of instances within the ASG.

### 6.2 Load balancing
To test the capablity of the application load balancer, I repeatedly used:
```shell
curl http://[ALB DNS]
```
A better was of doing this is using the following script to carry out the command 50 times within PowerShell
```shell
for ($i=0; $i -lt 50; $i++) {curl http://<ALB-DNS>}
```

This process requests information from the DNS address via the available port (HTTP Port 80) via the application load balancer. The expected behaviour is for the ALB to respond with:
```bash
echo "Hello from $(hostname)"
```
If the ALB is working I will receive greeting messages from instances with different IP addresses. </br>
Hello from ip 10.0.x.x </br>
Hello from ip 10.0.y.y


This test succeeded. I received responses from </br>
- 10.0.1.124 (Instance was assigned to public subnet to allow connectivity to install nginx, hence unexpected ip address.)
- 10.0.20.45

This test also confirmed onwards 
# 7.0 Troubleshooting and notes

## 7.1 SSH Forwarding
In Powershell, before using 'ssh -A' for SSH agent forwarding, I need to ensure that the SSH agent service is running. I can do this by using the following:
```shell
ssh-add ".../[.pem]"
ssh-add -L
start-service ssh-agent
```

Then upon reconnecting to the bastion host, I will be able to SSH into the target host using agent forwarding. 
```shell
ssh -A -i ".../[.pem"] ec2-user@[Target instance IP address]
```

And then onward connect to the next target host.
```shell
ssh ec2-user@[Target instance IP address]
```

Both hosts must be using the same key pair for agent forwarding to work. Otherwise, I would need to have the key somewhere accessible to the bastion host instead of using agent forwarding.

## 7.2 Target Groups and ALBs
Target Group for this lab needs to be the EC2 target group, not the VPC lattice target group.

## 7.3 Private instance connectivity
Instances launched by the Auto Scaling Group were being created successfully within the private subnets but were consistently marked as unhealthy in the target group.

Initial I reviews inbound access rules (SSH and HTTP), however the root cause was not related to security groups. Instead, the issue was due to a lack of outbound internet connectivity from the private subnets.

The launch template relied on 'user data' to install and configure nginx at instance startup.

Because the private subnets had no route to the internet (no NAT Gateway configured), instances were unable to reach external package repositories. As a result, nginx was not installed, no service was listening on port 80, and ALB health checks failed. 

I remediated this by creating a NAT gateway and updating the private route table 'sysop-private-rt' to inculde
```
0.0.0.0/0 --> NAT Gateway
```

Following this fix, 
- Instances successfully executed user data at launch
- nginx was installed and started correctly
- Target group health checks passed
- Instances transitioned to a healthy state