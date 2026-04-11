# Data Resilience and Backup

## Objective
Deploy a Multi-AZ Amazon RDS instance and validate backup and recovery mechanisms using EBS snapshots. This lab demonstrates how to ensure data availability, durability, and recoverability across both managed database services and underlying storage.

## Key concepts demonstrated:
- Amazon RDS Multi-AZ deployment for high availability
- Database subnet group configuration across multiple Availability Zones
- Private database placement within VPC subnets
- Security group chaining for EC2-to-RDS connectivity
- RDS failover behaviour and managed service resilience
- Automated backups vs manual snapshot strategies (RDS)
- EBS volume snapshot creation and lifecycle management
- Point-in-time recovery concepts for block storage
- Volume restoration from snapshots and reattachment to EC2
- Separation of compute and data layers in cloud architecture
- Differences between high availability and backup strategies
- Validation of recovery workflows and data persistence

## 1.0 Subnet groups and Security groups
### Subnet Groups
I created an RDS subnet group 'sysops-rds-subnet-group' within the existing 'sysops-lab-vpc'.

I added my already existing private subnets to the subnet group.
```shell
name: sysops-private-az1
availability-zone: eu-west-1a
CIDR: 10.0.10.0/24
```

```shell
name: sysops-private-az2
availability-zone: eu-west-1a
CIDR: 10.0.20.0/24
```
These subnets will be used for active and standby database instances placed by RDS.

### Security Group
Created new security group for RDS. The security group was configured for MySQL with the following configurations
```shell
name: sysops-rds-sg
VPC: sysops-lab-vpc

Inbound: TCP 3306 
```
Inbound traffic to the RDS will be TCP traffic via an instance within a new security group I created to hold a bastion host that I will access via SSH.


## RDS Instance

Configuration:
```shell
Engine: MySQL
ID: sysops-rds-db
MasterUsername: admin

```
Connectivity:
```shell
VPC: sysops-lab-vpc
DB subnet group: sysops-rds-subnet-group
Public access: No
VPC security group: sysops-rds-sg
```
Availability & Durability:
Multi-AZ deployment: Enabled

In the console, I confirmed that a standby instance was provisioned in the second availability zone and an RDS endpoint was generated.

### Connectivity testing
```shell
curl -o global-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
```

## Backup
## Create file
Create file in EC2 instance EBS:
```shell
echo "snapshot test $(date)" | sudo tee /tmp/snapshot-test.txt
cat /tmp/snapshot-test.txt
```

## Snapshots and Volumes

I identified the source volume by identifying that was attached to the target EC2 instance and recoreded the ID.
```shell
EBS Volumee ID: vol-060cd0741e8b7a101
echo "snapshot test $(date)" | sudo tee /tmp/snapshot-test.txt
cat /tmp/snapshot-test.txt
```

In the console, I created a snapshot of the volume with description 'sysops-ebs-test-snapshot'. I made sure that the Availability Zone matched the EC2 instance and inherited volume settings from the snapshot and attached the new volume to the EC2 instance.

I used the following code to list the contents of the volume. I had to ensure that this included the newly attached volume as well as the "default" EBS volume that the EC2 instance was launched with.
```shell
lsblk
sudo mkdir -p /mnt/restore
sudo mount /dev/xvdf /mnt/restore
```
I then verified that the test file exists on the restored volume:
```shell
ls -la /mnt/restore
cat /mnt/restore/tmp/snapshot-test.txt
```
Confirmed that the file contents matched the original data created prior to snapshot.

Once completed, all lab resources were terminated.