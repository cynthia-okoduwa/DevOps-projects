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
##### Note
A **terraform.tfstate** is created. Terraform uses this file to stay up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.
Another file **terraform.tfstate.lock.info** is also created in this process, but gets deleted immediately. Terraform uses it to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same – it allows to avoid duplicates and conflicts.

### Create Subnets
According to our architectural design, we require 6 subnets: 2 public, 2 private for webservers and 2 private for data layer. In this project we will be creating the public subnet only and the other 4 in subsequent projects.
1. In the main.tf file, add the following configuration below. We are declaring 2 resource blocks – one for each of the subnets. We are also using the **vpc_id** argument to interpolate the value of the VPC id by setting it to **aws_vpc.main.id**. This way, Terraform knows inside which VPC to create the subnet. 
```
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1b"
}
```

2. Run `terraform plan` to preview your configuration and `terraform apply` to create.
##### Note 
The above configurations has serveral problems that includes:
- Hard coded values: Both the availability_zone and cidr_block arguments are hard coded. We should always endeavour to make our work dynamic.
- Multiple Resource Blocks: we declared multiple resource blocks for each subnet in the code. This is bad coding practice. Best parctice is to create a single resource block that can dynamically create resources. If we had to create a lot of subnets, our code would quickly become overwelhming. To optimize this, we can make use of a count argument.
