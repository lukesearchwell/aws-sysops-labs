# Network Architecture


```mermaid

    flowchart LR
    
    
    
    subgraph AWS [AWS Account - eu-west-1]
        subgraph VPC [sysops-lab-vpc </br>10.0.0.0/16]
        direction TB
        
            IGW[Internet Gateway] 
            subgraph Public [Public Subnet 10.0.1.0/24]
            EC2Bastion[EC2 Bastion <br/>]
            end

            subgraph Private [Private Subnet 10.0.2.0/24]
            EC2App[EC2 App Instance]
            end
        end
    end
    
    subgraph Admin [Admin]
        direction TB

    end
   
    Internet -->IGW
    IGW -->| via Public Route table| EC2Bastion
    EC2Bastion-->|SSH via Private Route table |EC2App
    Admin -->|SSH|EC2Bastion
    Public -.- Private
```


## Route tables
### Public 
|Subnet | Route type |Route |
| --- | --- | --- |
| Private | local | 10.0.0.0/16 
| Public| sysops-igw | 0.0.0.0/0


### Private
|Subnet | Route type |Route |
| --- | --- | --- |
| Private | local |10.0.0.0/16 