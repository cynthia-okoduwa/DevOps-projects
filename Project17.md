## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 2
In continuation to [Project16](https://github.com/cynthia-okoduwa/DevOps-projects/blob/main/project16.md), in this project I created 
1. For Networking, in addition to the VPC and subnets created in [Project16](https://github.com/cynthia-okoduwa/DevOps-projects/blob/main/project16.md):
- Internet gateway
- Nat gateway
- Elastic IP, then allocate it to the Nat gateway
- Routes for both the public and private subnets
- Route tables both the public and private subnets
- Route table associations 
2. For Identity and Access Management
- IAM Roles for our instances to have access to certain resources
- IAM Policies to be attached to the roles
3. Other resouces to be created include:
- Security Groups
- Target Group for Nginx, WordPress and Tooling
- Certificate from AWS certificate manager
- External Application Load Balancer
- Internal Application Load Balancer
- Launch template for Bastion, Tooling, Nginx and WordPress
- Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress
- Elastic Filesystem
- Relational Database (RDS)

#### Internet Gateways & format() function
Create an Internet Gateway in a separate Terraform file internet_gateway.tf
