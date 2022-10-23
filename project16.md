## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1
### VPC | SUBNETS | SECURITY GROUPS
Set up Terraform CLI as per this instruction.
1. In Visual Studio Code: Create a folder called **PBL** Create a file in the folder, name it **main.tf**
2. Create a resource by declaring AWS as a provider, and a resource to create a VPC in the **main.tf** file. (Provider blocks inform Terraform 
that you intend to build infrastructure within AWS and resource block will create the resource you specified, in the case the VPC.)
```
provider "aws" {
  region = "eu-central-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}
```
3. Next, download the necessary plugins for Terraform to work. These plugins are used by **Providers** and **Provisioners**.To accomplish this
run `terraform init` command.
4. A new directory is created: **.terraform\....** This is where Terraform keeps plugins. Generally, it is safe to delete this folder. 
It just means that you must execute terraform init again, to download them.
5. Next, create the resource we just defined: **aws_vpc**. But before that, check to see what terraform intends to create before we tell it to 
go ahead and create it. Run: `terraform plan`
6.If you are happy with changes planned, execute `terraform apply`
