## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 3
#### REFACTOR YOUR PROJECT USING MODULES

1. In the previous project our code was wriiten in a long list of files that created our resources. Even though it worked, this is however is not the best 
way to manage your code as this would make your code diifcult to understand and future changes very difficult. In this project we would start with refactoring
the project using Modules. A module allows you to create logical abstraction on the top of some resource set. In other words, a module allows you 
to group resources together and reuse this group later, possibly many times.
2. We start with breaking down the Terraform codes to have all resources in their respective modules. Combine resources of a similar type into directories 
within a module. Create a new directory and name it `modules` inside the new directory, create modules that would be used to organise your resources like the list
is below:
```
- modules
  - ALB (For Apllication Load balancer and similar resources)
  - EFS (For Elastic file system resources)
  - RDS (For Databases resources)
  - Autoscaling (For Autosacling and launch template resources)
  - compute (For EC2 and rlated resources)
  - VPC (For VPC and netowrking resources such as subnets, roles, e.t.c.)
  - security (For creating security group resources)
```
3. Each Module should have 2 more of the following files:

- **main.tf** (or %resource_name%.tf) file(s) with resources blocks
- **outputs.tf** (optional, if you need to refer outputs from any of these resources in your root module)
- **variables.tf** (as we learned before - it is a good practice not to hard code the values and use variables)
4. In the ALB module create the following files:
- alb.tf 
- cert.tf 
- output.tf
- variables.tf (To declare variable that are created within the module)
5. In the autoscaling module create:
- asg-bastion-nginx.tf
- asg-webserever.tf
- lt-bastion-nginx.tf
- lt-tooling-wp.tf
- bastion.sh
- nginx.sh
- tooling.sh
- wordpress.sh
- variables.tf
6. In the compute module, create the following:
- main.tf
- variables.tf
7. Create the following in the EFS module:
- efs.tf
- variables.tf
8. Create the following in the RDS module:
- rds.tf
- variables.tf
9. For security module, craete:
- main.tf
- output.tf
- variables.tf
- security.tf
- sg-rule.tf
10. Finally in VPC module, create:
- internet-gateway.tf
- natgateway.tf
- main.tf
- outputs.tf
- role.tf
- route-tables.tf
11. Finally in the root module, create the following files:
- main.tf
- variables.tf
- terraform.tfvars
- providers.tf
- backend.tf
12. 
