# EC2 + IAM Lab Notes

## EC2 Launch – Console

Instance Name: sysops-lab-ec2-console

Instance ID: 0460888c7ca9c84fc
Public IPv4: 108.130.124.14
VPC ID: vpc-098ae3f78bca20304
Subnet ID: subnet-035f8bad2719d3250
Security Group: sg-009999c49ef929acf

Notes: EC2 Instance. Configured within a DEFAULT VPC. Security Group configured to only allow connectioned from my IP address.

## EC2 Launch – CLI

Instance Name: sysops-lab-ec2-cli  
Instance ID: i-0d5e1766e99478514  
Instance Type: t3.micro  
AMI ID: ami-008f3b045fbd24779  
Key Pair: sysops-lab-ec2-key  
Security Group: launch-wizard-1  
Public IPv4: 54.216.104.38  
Region: eu-west-1

Notes: EC2 Instance launched via Powershell. Confirmed instance state and config. Used "aws ec2-describe-images...--owners amazon" to choose the most up to date ami. Used filters to only show AMI's for Amazon Linux 2023 x86_64. EC2 deployment via CLI enables repeatable infrastructure as code deployments. When spinning up an individual instance, this workflow route may be more time consuming than using the Amazon console. 

## IAM Role + IMDS Verification

Role name returned from IMDS: ec2-s3-read-role

Temporary credentials retrieved successfully from IMDSv2.
Observed fields:
- AccessKeyId
- SecretAccessKey
- Token
- Expiration

Notes:
- IAM role applied to EC2 instance to allow read only access to s3 buckets.
- No static credentials were configured on the EC2 instance.
- `aws s3 ls` succeeded using temporary role-based credentials.
- IMDSv2 token flow was required to query metadata manually.
- Issue: Plain "curl" was not working so token request was required to get temporary credentials.
- Issue: IAM Role 'AmazonS3ReadOnlyAccess' includes "s3:list*" which meant that permissions to list all buckets and objects stored in the s3 bucket was granted. I detached the IAM policy and confirmed access denied for EC2 "sysops-lab-ec2-cli" when executing 'aws s3 ls' 
