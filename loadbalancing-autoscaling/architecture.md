# Network Architecture


```mermaid

    flowchart LR
    
    
    
    subgraph AWS [AWS Account - eu-west-1]
        NAT[NAT Gateway]
        CASP[CloudWatch Alarm/Scaling Policy]
            subgraph VPC [sysops-lab-vpc </br>10.0.0.0/16]
            IGW[Internet Gateway] 
                subgraph Public [Public Subnet AZ-a, AZ-b]
                    ALB[Application Load Balancer]
                    end
                TG[Target Group]
                subgraph Private [Private Subnet]
                subgraph WA1 [EC2 Web/App 1 Private Subnet AZ-a]
                    EC2App1[EC2 Web App]
                    end
                subgraph WA2 [EC2 Web/App 2 Private Subnet AZ-b]
                    EC2App2[EC2 Web App]
                    end
            end
                ASG[Auto Scaling Group]
            end
    end     
    
    Internet --> IGW
    IGW --> Public
    Public --> TG
    TG --> WA1
    TG --> WA2
    WA1 --> ASG
    WA2 --> ASG
    ASG <--> CASP
    Internet --> NAT
    NAT -.- Private