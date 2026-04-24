# Terraform VPC
## Objective: 
Design and implement a reusable Terraform module to provision a custom VPC, enabling consistent, scalable, and modular infrastructure deployment.


### Key concepts demonstrated:
- Terraform module design and structure
- Reusable Infrastructure as Code (IaC) patterns
- Dependency management between resources
- Transformation of AWS networking components into code
- Abstraction of complex infrastructure into reusable units

Successfully created VPC infrastructure in AWS using terraform:
```shell
Apply complete! Resources: 13 added, 0 changed, 0 destroyed.

Outputs:

internet_gateway_id = "igw-09f09026ddfee5de3"
private_subnet_ids = [
  "subnet-0ad9a520b8d7a4776",
  "subnet-0fedc6d7034188c0d",
]
public_subnet_ids = [
  "subnet-0da49e164c81a5422",
  "subnet-04fadd5313660a69d",
]
vpc_id = "vpc-09b02cb0275c3deac"
```


## Troubleshooting
`terraform init` and `terraform validate` were working as expected, but upon using `terraform plan`, I received multiple errors. 

### 1. Invalid credentials 
I received the following error message when attempting to connect to aws using an existing profile and secret via `aws configure --profile sysops-lab`. 
```
aws: [ERROR]: An error occurred (InvalidClientTokenId) when calling the GetCallerIdentity operation: The security token included in the request is invalid.

Additional error details:
Type: Sender
```
To solve this issue, rather than using a key to gain access, I set up the AWS IAM identity Center and created a new identity to allow me to gain sso access to aws.

Setting this ip invalidates my need for an access token. Rather, via the terminal each time that I want to gain administrator access I will use `aws configure sso` and log into my new profile which is alsoprotected by MFA.


### 2. Terraform "Invalid provider configuration"
When attempting to run `terraform plan`, I received the following error
```
Error: Failed to query available provider packages

Could not retrieve the list of available versions for provider hashicorp/aws: locked provider registry.terraform.io/hashicorp/aws 6.41.0 does not match configured version constraint ~> 5.0; must use terraform init -upgrade to allow selection of new versions.
```

Within `main.tf`. the block which I used to specify the provider was the following
```shell
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
        source = "hashicorp/aws"
        version = "~> 5.0"
    }
  }
}
```
I amended the version specified in `required_providers` to 6.0 and this fixed the issue.

```shell
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
        source = "hashicorp/aws"
        version = "~> 6.0"
    }
  }
}
```


