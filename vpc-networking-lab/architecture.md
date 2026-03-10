```mermaid

architecture-beta
    %% Groups
    group 1 [AWS Region euwest1]

    group VPC [SysOps Lab VPC] in 1
    group Private(icon:aws:private-subnet) [Private Subnet] in VPC
    group Public(icon:aws:public-subnet) [Public Subnet] in VPC
    group edge(cloud) [Edge Services] in 1
        
    %%Services
    service EC2 (icon:aws:res-amazon-ec2-instance) [EC2 App Instance] in Private
    service EC2Bastion (icon:aws:res-amazon-ec2-instance) [EC2 Bastion Host] in Public
    service Gateway (icon:aws:res-amazon-vpc-internet-gateway) [Internet Gateway] in edge
    service Internet (icon:aws:res-globe) [Internet]

    %%Links
    Gateway:L -- R:EC2Bastion
    EC2Bastion:L -- R:EC2
    Internet:B -- T:Gateway