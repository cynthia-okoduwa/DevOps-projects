## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 3
#### REFACTOR YOUR PROJECT USING MODULES

1. In the previous project our code was wriiten in a long list of files that created our resources. Even though it worked, this is however is not the best 
way to manage your code as this would make your code diifcult to understand and future changes very difficult. In this project we would start with refactoring
the project using Modules. A module allows you to create logical abstraction on the top of some resource set. In other words, a module allows you 
to group resources together and reuse this group later, possibly many times.
2. We start with breaking down the Terraform codes to have all resources in their respective modules. Combine resources of a similar type into directories 
within a module. When making use of Modules in Terraform, it is important to declare your arguments as variables, then define those variables in the variables.tf file of the particular module you are working on.
3. Create a new directory and name it `modules` inside the new directory, create modules that would be used to organise your resources like the list
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
4. Next, we begin categorizing resources into their respective modules. In the ALB module create the following files:
- `alb.tf` - Place your code for the creation of your internat and external Application load balancers, Target groups, Listeners and rules here:
```
resource "aws_lb" "ext-alb" {
  name     = var.name
  internal = false
  security_groups = [var.public-sg]

  subnets = [
    var.public-subnet-1,
    var.public-subnet-2,
  ]

   tags = merge(
    var.tags,
    {
      Name = var.name
    },
  )

  ip_address_type    = var.ip_address_type
  load_balancer_type = var.load_balancer_type
}

# --- create a target group for the external load balancer

resource "aws_lb_target_group" "nginx-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }
  name        = "nginx-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = var.vpc_id
}

# --- create listener for load balancer

resource "aws_lb_listener" "nginx-listner" {
  load_balancer_arn = aws_lb.ext-alb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.bulwm.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}

# ----------------------------
#Internal Load Balancers for webservers
#---------------------------------

resource "aws_lb" "ialb" {
  name     = "ialb"
  internal = true
  security_groups = [var.private-sg]

  subnets = [var.private-subnet-1,
    var.private-subnet-2,]

  tags = merge(
    var.tags,
    {
      Name = "ACS-int-alb"
    },
  )

  ip_address_type    = var.ip_address_type
  load_balancer_type = var.load_balancer_type
}

# --- target group  for wordpress -------

resource "aws_lb_target_group" "wordpress-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "wordpress-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = var.vpc_id
}

# --- target group for tooling -------

resource "aws_lb_target_group" "tooling-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "tooling-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = var.vpc_id
}

# For this aspect a single listener was created for the wordpress which is default,
# A rule was created to route traffic to tooling when the host header changes

resource "aws_lb_listener" "web-listener" {
  load_balancer_arn = aws_lb.ialb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.bulwm.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.wordpress-tgt.arn
  }
}

# listener rule for tooling target

resource "aws_lb_listener_rule" "tooling-listener" {
  listener_arn = aws_lb_listener.web-listener.arn
  priority     = 99

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tooling-tgt.arn
  }

  condition {
    host_header {
      values = ["tooling.bulwm.click"]
    }
  }
}
```
- `cert.tf` - This is where you create a certificate, public zone, and validate the certificate using DNS method
``` 
# The entire section create a certiface, public zone, and validate the certificate using DNS method

# Create the certificate using a wildcard for all the domains created in bulwm.click
resource "aws_acm_certificate" "bulwm" {
  domain_name       = "*.bulwm.click"
  validation_method = "DNS"
}

# calling the hosted zone
data "aws_route53_zone" "bulwm" {
  name         = "bulwm.click"
  private_zone = false
}

# selecting validation method
resource "aws_route53_record" "bulwm" {
  for_each = {
    for dvo in aws_acm_certificate.bulwm.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.bulwm.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "bulwm" {
  certificate_arn         = aws_acm_certificate.bulwm.arn
  validation_record_fqdns = [for record in aws_route53_record.bulwm : record.fqdn]
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.bulwm.zone_id
  name    = "tooling.bulwm.click"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}

# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.bulwm.zone_id
  name    = "wordpress.bulwm.click"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}
```
- `output.tf` - This would output
```
output "alb_dns_name" {
  value       = aws_lb.ext-alb.dns_name
  description = "External load balance arn"
}

output "nginx-tgt" {
  description = "External Load balancaer target group"
  value       = aws_lb_target_group.nginx-tgt.arn
}


output "wordpress-tgt" {
  description = "wordpress target group"
  value       = aws_lb_target_group.wordpress-tgt.arn
}


output "tooling-tgt" {
  description = "Tooling target group"
  value       = aws_lb_target_group.tooling-tgt.arn
}
```
- `variables.tf` - Use this to define the variables that were decalred within the module:
```
# Security group for external loadbalancer
variable "public-sg"{
    description = "Security group for external load balancer"
}

# Public subnets for external loadbalancer
variable "public-subnet-1"{
    description = "Public subnets tor deploy external ALB"
}
variable "public-subnet-2"{
    description = "Public subnets tor deploy external ALB"
}

# VPC id variable
variable "vpc_id"{
    type = string
    description = "The VPC ID"
}

# Security group for internal loadbalancer
variable "private-sg"{
    description = "Security group for internal loadbalancer"
}

# Private subnets for internal loadbalancer
variable "private-subnet-1"{
    description = "Private subnets tor deploy internal ALB"
}
variable "private-subnet-2"{
    description = "Private subnets tor deploy external ALB"
}

# ALB IP address variable
variable "ip_address_type"{
    type = string
    description = "IP address for the ALB"
}

# Loadbalancer type variable
variable "load_balancer_type"{
    type = string
    description = "The type of load balancer"
}

# Tags variable
variable "tags"{
    description = "A mapping of tags to assign to all resources"
    type = map(string)
    default = {}
}

variable "name"{
    type = string
    description = "name of the loadbalancer"
}
```
5. In the autoscaling module create:
- `asg-bastion-nginx.tf` - Create SNS topics, notification and autoscaling for both nginx and bastion here. Don't forget to delare your variables where necessary:
```
# Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}


# creating sns topic for all the auto scaling groups
resource "aws_sns_topic" "ACS-sns" {
  name = "Default_CloudWatch_Alarms_Topic"
}


# creating notification for all the auto scaling groups
resource "aws_autoscaling_notification" "david_notifications" {
  group_names = [
    aws_autoscaling_group.bastion-asg.name,
    aws_autoscaling_group.nginx-asg.name,
    aws_autoscaling_group.wordpress-asg.name,
    aws_autoscaling_group.tooling-asg.name,
  ]
  notifications = [
    "autoscaling:EC2_INSTANCE_LAUNCH",
    "autoscaling:EC2_INSTANCE_TERMINATE",
    "autoscaling:EC2_INSTANCE_LAUNCH_ERROR",
    "autoscaling:EC2_INSTANCE_TERMINATE_ERROR",
  ]

  topic_arn = aws_sns_topic.ACS-sns.arn
}


resource "random_shuffle" "az_list" {
  input = data.aws_availability_zones.available.names
}


# ---- Autoscaling for bastion  hosts


resource "aws_autoscaling_group" "bastion-asg" {
  name                      = "bastion-asg"
  max_size                  = var.max_size
  min_size                  = var.min_size
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = var.desired_capacity

  vpc_zone_identifier = var.public_subnets
  



  launch_template {
    id      = aws_launch_template.bastion-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "ACS-bastion"
    propagate_at_launch = true
  }

}

# ------ Autoscslaling group for reverse proxy nginx ---------

resource "aws_autoscaling_group" "nginx-asg" {
  name                      = "nginx-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1
  
  vpc_zone_identifier = var.public_subnets


  launch_template {
    id      = aws_launch_template.nginx-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "ACS-nginx"
    propagate_at_launch = true
  }


}

# attaching autoscaling group of nginx to external load balancer
resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
  autoscaling_group_name = aws_autoscaling_group.nginx-asg.id
  alb_target_group_arn   = var.nginx-alb-tgt
}
```
- `asg-webserever.tf` - Create autoscaling for the Wordpress and Tooling applications slao making sure that variables are used in place of hard-coded values:
```
# ---- Autoscaling for wordpress application

resource "aws_autoscaling_group" "wordpress-asg" {
  name                      = "wordpress-asg"
  max_size                  = var.max_size
  min_size                  = var.min_size
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = var.desired_capacity
  vpc_zone_identifier = var.private_subnets


  launch_template {
    id      = aws_launch_template.wordpress-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "ACS-wordpress"
    propagate_at_launch = true
  }
}


# attaching autoscaling group of  wordpress application to internal loadbalancer
resource "aws_autoscaling_attachment" "asg_attachment_wordpress" {
  autoscaling_group_name = aws_autoscaling_group.wordpress-asg.id
  alb_target_group_arn   = var.wordpress-alb-tgt
}

# ---- Autoscaling for tooling -----

resource "aws_autoscaling_group" "tooling-asg" {
  name                      = "tooling-asg"
  max_size                  = var.max_size
  min_size                  = var.min_size
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = var.desired_capacity
  vpc_zone_identifier = var.private_subnets

  launch_template {
    id      = aws_launch_template.tooling-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "ACS-tooling"
    propagate_at_launch = true
  }
}

# attaching autoscaling group of  tooling application to internal loadbalancer
resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
  autoscaling_group_name = aws_autoscaling_group.tooling-asg.id
  alb_target_group_arn   = var.tooling-alb-tgt
}
```
- `lt-bastion-nginx.tf` Create Launch templates for bastion and ngnix
```
# launch template for bastion

resource "aws_launch_template" "bastion-launch-template" {
  image_id               = var.ami-bastion
  instance_type          = "t2.micro"
  vpc_security_group_ids = var.bastion-sg

  iam_instance_profile {
    name = var.instance_profile
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"


  tags = merge(
    var.tags,
    {
      Name = "bastion-launch-template"
    },
  )
    
  }

  user_data = filebase64("${path.module}/bastion.sh")
}


# launch template for nginx

resource "aws_launch_template" "nginx-launch-template" {
  image_id               = var.ami-nginx
  instance_type          = "t2.micro"
  vpc_security_group_ids = var.nginx-sg

  iam_instance_profile {
    name = var.instance_profile
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

  tags = merge(
    var.tags,
    {
      Name = "nginx-launch-template"
    },
  )
  }

  user_data = filebase64("${path.module}/nginx.sh")
}
```
- `lt-tooling-wp.tf` - Launch template for tooling and wordpress applications: 
```
# launch template for wordpress

resource "aws_launch_template" "wordpress-launch-template" {
  image_id               = var.ami-web
  instance_type          = "t2.micro"
  vpc_security_group_ids = var.web-sg

  iam_instance_profile {
    name = var.instance_profile
  }

  key_name = var.keypair


  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

  tags = merge(
    var.tags,
    {
      Name = "wordpress-launch-template"
    },
  )
  }

  user_data = filebase64("${path.module}/wordpress.sh")
}


# launch template for toooling
resource "aws_launch_template" "tooling-launch-template" {
  image_id               = var.ami-web
  instance_type          = "t2.micro"
  vpc_security_group_ids = var.web-sg

  iam_instance_profile {
    name = var.instance_profile
  }

  key_name = var.keypair


  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }
  

  tag_specifications {
    resource_type = "instance"


 tags = merge(
    var.tags,
    {
      Name = "tooling-launch-template"
    },
  )
  }

  user_data = filebase64("${path.module}/tooling.sh")
}
```
- `bastion.sh` - same as [project 17](https://github.com/darey-devops/PBL-project-17)
- `nginx.sh` - same as [project 17](https://github.com/darey-devops/PBL-project-17)
- `tooling.sh` - same as [project 17](https://github.com/darey-devops/PBL-project-17)
- `wordpress.sh` - same as [project 17](https://github.com/darey-devops/PBL-project-17)
- `variables.tf` - All the variables you declared within the module is to be defined here:
```
variable "ami-web" {
  type        = string
  description = "ami for webservers"
}

variable "instance_profile" {
  type        = string
  description = "Instance profile for launch template"
}


variable "keypair" {
  type        = string
  description = "Keypair for instances"
}

variable "ami-bastion" {
  type        = string
  description = "ami for bastion"
}

variable "web-sg" {
  type = list
  description = "security group for webservers"
}

variable "bastion-sg" {
  type = list
  description = "security group for bastion"
}

variable "nginx-sg" {
  type = list
  description = "security group for nginx"
}

variable "private_subnets" {
  type = list
  description = "first private subnets for internal ALB"
}


variable "public_subnets" {
  type = list
  description = "Seconf subnet for ecternal ALB"
}


variable "ami-nginx" {
  type        = string
  description = "ami for nginx"
}

variable "nginx-alb-tgt" {
  type        = string
  description = "nginx reverse proxy target group"
}

variable "wordpress-alb-tgt" {
  type        = string
  description = "wordpress target group"
}


variable "tooling-alb-tgt" {
  type        = string
  description = "tooling target group"
}


variable "max_size" {
  type        = number
  description = "maximum number for autoscaling"
}

variable "min_size" {
  type        = number
  description = "minimum number for autoscaling"
}

variable "desired_capacity" {
  type        = number
  description = "Desired number of instance in autoscaling group"

}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}
```
6. In the compute module, create the following:
- `main.tf` - Create your instances for Sonarqube, Artifactory and Jenkins
```
# create instance for jenkins
resource "aws_instance" "Jenkins" {
  ami                         = var.ami-jenkins
  instance_type               = "t2.micro"
  subnet_id                   = var.subnets-compute
  vpc_security_group_ids      = var.sg-compute
  associate_public_ip_address = true
  key_name                    = var.keypair

 tags = merge(
    var.tags,
    {
      Name = "ACS-Jenkins"
    },
  )
}


#create instance for sonbarqube
resource "aws_instance" "sonbarqube" {
  ami                         = var.ami-sonar
  instance_type               = "t2.medium"
  subnet_id                   = var.subnets-compute
  vpc_security_group_ids      = var.sg-compute
  associate_public_ip_address = true
  key_name                    = var.keypair


   tags = merge(
    var.tags,
    {
      Name = "ACS-sonbarqube"
    },
  )
}

# create instance for artifactory
resource "aws_instance" "artifactory" {
  ami                         = var.ami-jfrog
  instance_type               = "t2.medium"
  subnet_id                   = var.subnets-compute
  vpc_security_group_ids      = var.sg-compute
  associate_public_ip_address = true
  key_name                    = var.keypair


  tags = merge(
    var.tags,
    {
      Name = "ACS-artifactory"
    },
  )
}
```
- `variables.tf` - To define the variables that are decalred within the compute module:
```
variable "subnets-compute" {
    description = "public subnetes for compute instances"
}
variable "ami-jenkins" {
    type = string
    description = "ami for jenkins"
}
variable "ami-jfrog" {
    type = string
    description = "ami for jfrob"
}
variable "ami-sonar" {
    type = string
    description = "ami foir sonar"
}
variable "sg-compute" {
    description = "security group for compute instances"
}
variable "keypair" {
    type = string
    description = "keypair for instances"
}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}
```
7. Create the following in the EFS module:
- `efs.tf` - create key for Key management system, EFS and every other related resource here, declaring variables where neccessary:
```
# create key from key management system
resource "aws_kms_key" "ACS-kms" {
  description = "KMS key"
  policy      = <<EOF
  {
  "Version": "2012-10-17",
  "Id": "kms-key-policy",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::${var.account_no}:user/terraform" },
      "Action": "kms:*",
      "Resource": "*"
    }
  ]
}
EOF
}

# create key alias
resource "aws_kms_alias" "alias" {
  name          = "alias/kms"
  target_key_id = aws_kms_key.ACS-kms.key_id
}

# create Elastic file system
resource "aws_efs_file_system" "ACS-efs" {
  encrypted  = true
  kms_key_id = aws_kms_key.ACS-kms.arn

  tags = merge(
    var.tags,
    {
      Name = "ACS-efs"
    },
  )
}

# set first mount target for the EFS 
resource "aws_efs_mount_target" "subnet-1" {
  file_system_id  = aws_efs_file_system.ACS-efs.id
  subnet_id       = var.efs-subnet-1
  security_groups = var.efs-sg
}

# set second mount target for the EFS 
resource "aws_efs_mount_target" "subnet-2" {
  file_system_id  = aws_efs_file_system.ACS-efs.id
  subnet_id       = var.efs-subnet-2
  security_groups = var.efs-sg
}

# create access point for wordpress
resource "aws_efs_access_point" "wordpress" {
  file_system_id = aws_efs_file_system.ACS-efs.id

  posix_user {
    gid = 0
    uid = 0
  }

  root_directory {
    path = "/wordpress"

    creation_info {
      owner_gid   = 0
      owner_uid   = 0
      permissions = 0755
    }

  }

}

# create access point for tooling
resource "aws_efs_access_point" "tooling" {
  file_system_id = aws_efs_file_system.ACS-efs.id
  posix_user {
    gid = 0
    uid = 0
  }

  root_directory {

    path = "/tooling"

    creation_info {
      owner_gid   = 0
      owner_uid   = 0
      permissions = 0755
    }

  }
}
```
- `variables.tf` - To define the variables that are decalred within the module:
```
variable "efs-subnet-2" {
  description = "Second subnet for the mount target"
}

variable "efs-subnet-1" {
  description = "First subnet for the mount target"
}

variable "efs-sg" {
  type        = list
  description = "security group for the file system"

}

variable "account_no" {
  type        = string
  description = "account number for the aws"
} 


variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}
```
8. Create the following in the RDS module:
- `rds.tf` - Creat RDS instance
```
# This section will create the subnet group for the RDS  instance using the private subnet
resource "aws_db_subnet_group" "ACS-rds" {
  name       = "acs-rds"
  subnet_ids = var.private_subnets

 tags = merge(
    var.tags,
    {
      Name = "ACS-database"
    },
  )
}

# create the RDS instance with the subnets group
resource "aws_db_instance" "ACS-rds" {
  allocated_storage      = 20
  storage_type           = "gp2"
  engine                 = "mysql"
  engine_version         = "5.7"
  instance_class         = "db.t2.micro"
  db_name                = "cynthiadb"
  username               = var.master-username
  password               = var.master-password
  parameter_group_name   = "default.mysql5.7"
  db_subnet_group_name   = aws_db_subnet_group.ACS-rds.name
  skip_final_snapshot    = true
  vpc_security_group_ids = var.db-sg
  multi_az               = "true"
}
```
- `variables.tf` - To define the variables that are decalred within the RDS module:
```
variable "master-username" {
  type        = string
  description = "The master user name"
}


variable "master-password" {
  type        = string
  description = "Master password"
}

variable "db-sg" {
  type = list
  description = "The DB security group"
}

variable "private_subnets" {
  type        = list
  description = "Private subnets fro DB subnets group"
}


variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}
```
9. For security module, craete:
- `main.tf` - For repetitive blocks of code you can use **dynamic blocks** in Terraform. Copy the following to create security groups dynamically
```
# create all security groups dynamically
resource "aws_security_group" "ACS" {
  for_each    = local.security_groups
  name        = each.value.name
  description = each.value.description
  vpc_id      = var.vpc_id

 
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = each.value.name
    },
  )
}
```
- `security.tf` - Create security groups for bastion, nginx, webservers. etc.
```
# creating dynamic ingress security groups
locals {
  security_groups = {
    ext-alb-sg = {
      name        = "ext-alb-sg"
      description = "for external loadbalncer"

    }

    # security group for bastion
    bastion-sg = {
      name        = "bastion-sg"
      description = "for bastion instances"
    }

    # security group for nginx
    nginx-sg = {
      name        = "nginx-sg"
      description = "nginx instances"
    }

    # security group for IALB
    int-alb-sg = {
      name        = "int-alb-sg"
      description = "IALB security group"
    }


    # security group for webservers
    webserver-sg = {
      name        = "webserver-sg"
      description = "webservers security group"
    }


    # security group for data-layer
    datalayer-sg = {
      name        = "datalayer-sg"
      description = "data layer security group"


    }
  }
}
```
- `sg-rule.tf` - Here you create the security group rules:
```
# security group for alb, to allow acess from any where on port 80 for http traffic
resource "aws_security_group_rule" "inbound-alb-http" {
  from_port         = 80
  protocol          = "tcp"
  to_port           = 80
  type              = "ingress"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.ACS["ext-alb-sg"].id
}
 

resource "aws_security_group_rule" "inbound-alb-https" {
  from_port         = 443
  protocol          = "tcp"
  to_port           = 443
  type              = "ingress"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.ACS["ext-alb-sg"].id
}

# security group rule for bastion to allow assh access fro your local machine
resource "aws_security_group_rule" "inbound-ssh-bastion" {
  from_port         = 22
  protocol          = "tcp"
  to_port           = 22
  type              = "ingress"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.ACS["bastion-sg"].id
}

# security group for nginx reverse proxy, to allow access only from the extaernal load balancer and bastion instance

resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ACS["ext-alb-sg"].id
  security_group_id        = aws_security_group.ACS["nginx-sg"].id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ACS["bastion-sg"].id
  security_group_id        = aws_security_group.ACS["nginx-sg"].id
}

# security group for ialb, to have acces only from nginx reverser proxy server

resource "aws_security_group_rule" "inbound-ialb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ACS["nginx-sg"].id
  security_group_id        = aws_security_group.ACS["int-alb-sg"].id
}

# security group for webservers, to have access only from the internal load balancer and bastion instance

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ACS["int-alb-sg"].id
  security_group_id        = aws_security_group.ACS["webserver-sg"].id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ACS["bastion-sg"].id
  security_group_id        = aws_security_group.ACS["webserver-sg"].id
}

# security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port
resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ACS["webserver-sg"].id
  security_group_id        = aws_security_group.ACS["datalayer-sg"].id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ACS["bastion-sg"].id
  security_group_id        = aws_security_group.ACS["datalayer-sg"].id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ACS["webserver-sg"].id
  security_group_id        = aws_security_group.ACS["datalayer-sg"].id
}
```
- output.tf
```
output "ALB-sg" {
  value = aws_security_group.ACS["ext-alb-sg"].id
}

output "IALB-sg" {
  value = aws_security_group.ACS["int-alb-sg"].id
}

output "bastion-sg" {
  value = aws_security_group.ACS["bastion-sg"].id
}

output "nginx-sg" {
  value = aws_security_group.ACS["nginx-sg"].id
}

output "web-sg" {
  value = aws_security_group.ACS["webserver-sg"].id
}

output "datalayer-sg" {
  value = aws_security_group.ACS["datalayer-sg"].id
}
```
- `variables.tf` - To define the variables that are decalred within the module
```
variable "vpc_id" {
  type        = string
  description = "the vpc id"
}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}
```

10. Finally for the modules, in VPC module, create:
- `main.tf` - Create your VPC and Subnets both the private and publice subnets:
```

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support
  enable_dns_hostnames           = var.enable_dns_hostnames
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink_dns_support

tags = merge(
    var.tags,
    {
      Name = format("%s-VPC", var.name)
    },
  )

}

# Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Create public subnets
resource "aws_subnet" "public" {
  count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnets[count.index]
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

tags = merge(
    var.tags,
    {
      Name = format("%s-PublicSubnet-%s", var.name, count.index)
    },
  )

}

# Create private subnets
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.private_subnets[count.index]
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

tags = merge(
    var.tags,
    {
      Name = format("%s-PrivateSubnet-%s", var.name, count.index)
    },
  )

}
```
- `internet-gateway.tf` - Create your internet gateway here:
```
resource "aws_internet_gateway" "ig" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-%s!", aws_vpc.main.id,"IG")
    } 
  )
}
```
- `natgateway.tf` -Create Nat gateway here:
```
resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on    = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-Nat", var.name)
    },
  )
}
```
- `outputs.tf` -
```
output "public_subnets-1" {
  value       = aws_subnet.public[0].id
  description = "The first public subnet in the subnets"
}

output "public_subnets-2" {
  value       = aws_subnet.public[1].id
  description = "The first public subnet"
}

output "private_subnets-1" {
  value       = aws_subnet.private[0].id
  description = "The first private subnet"
}

output "private_subnets-2" {
  value       = aws_subnet.private[1].id
  description = "The second private subnet"
}

output "private_subnets-3" {
  value       = aws_subnet.private[2].id
  description = "The third private subnet"
}

output "private_subnets-4" {
  value       = aws_subnet.private[3].id
  description = "The fourth private subnet"
}

output "vpc_id" {
  value = aws_vpc.main.id
}

output "instance_profile" {
  value = aws_iam_instance_profile.ip.id
}
```
- `role.tf` - This is to create IAM roles, policy and other required resources:
```
# Create AssumeRole
resource "aws_iam_role" "ec2_instance_role" {
name = "ec2_instance_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      },
    ]
  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume role"
    },
  )
}

# Create IAM policy for the role created

resource "aws_iam_policy" "policy" {
  name        = "ec2_instance_policy"
  description = "A test policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:Describe*",
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]

  })

  tags = merge(
    var.tags,
    {
      Name =  "aws assume policy"
    },
  )

}

# Attach policy to IAM role
resource "aws_iam_role_policy_attachment" "test-attach" {
        role       = aws_iam_role.ec2_instance_role.name
        policy_arn = aws_iam_policy.policy.arn
    }

# Create an Instance Profile and interpolate the IAM Role

resource "aws_iam_instance_profile" "ip" {
        name = "aws_instance_profile_test"
        role =  aws_iam_role.ec2_instance_role.name
    }
```
- `route-tables.tf` - Here is where you create all your required route tables:
```
# create private route table
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Private-Route-Table", var.name)
    },
  )
}

# associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
  count          = length(aws_subnet.private[*].id)
  subnet_id      = element(aws_subnet.private[*].id, count.index)
  route_table_id = aws_route_table.private-rtb.id
}

# create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table", var.name)
    },
  )
}

# create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
  route_table_id         = aws_route_table.public-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.ig.id
}

# associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
  count          = length(aws_subnet.public[*].id)
  subnet_id      = element(aws_subnet.public[*].id, count.index)
  route_table_id = aws_route_table.public-rtb.id
}
```
- `variables.tf` - To define the variables that are decalred within the VPC module:
```
variable "region" {
}

variable "vpc_cidr" {
  type = string
}

variable "enable_dns_support" {
  type = bool
}

variable "enable_dns_hostnames" {
  type = bool
}

variable "enable_classiclink" {
  type = bool
}

variable "enable_classiclink_dns_support" {
  type = bool
}


variable "preferred_number_of_public_subnets" {
  type = number
}

variable "preferred_number_of_private_subnets" {
  type = number
}

variable "private_subnets" {
  type        = list(any)
  description = "List of private subnets"
}

variable "public_subnets" {
  type        = list(any)
  description = "list of public subnets"

}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}

variable "name" {
  type    = string
  default = "ACS"

}
variable "environment" {
  default = "true"
}
```
11. Finally let's work on the root module. Files placed in the root module have a one-on-one relationship with all the files in the module. this is where you make references to all the other modules. In the root module, create the following files:
- `main.tf` - This file is where you create Module Blocks to call modules in other directories. Start by declaring a module and give it the name of the module you are referencing. Next, give it a source (in this case the source is my local directory). Then for the body of the block, reference the variables you declared as an argument in the variable.tf files of the module as your paramenter. In some cases you may also need the output value of a particular module in another module (for exampe the ALB module needs the VPC ID from the VPC module), you can achieve this by calling the particular module you need, eg. `module.VPC.vpc_id` This is referencing the output of the VPC module.
```
# creating VPC
module "VPC" {
  source                              = "./modules/VPC"
  region                              = var.region
  vpc_cidr                            = var.vpc_cidr
  enable_dns_support                  = var.enable_dns_support
  enable_dns_hostnames                = var.enable_dns_hostnames
  enable_classiclink                  = var.enable_classiclink
  enable_classiclink_dns_support      = var.enable_classiclink_dns_support
  preferred_number_of_public_subnets  = var.preferred_number_of_public_subnets
  preferred_number_of_private_subnets = var.preferred_number_of_private_subnets
  private_subnets                     = [for i in range(1, 8, 2) : cidrsubnet(var.vpc_cidr, 8, i)]
  public_subnets                      = [for i in range(2, 5, 2) : cidrsubnet(var.vpc_cidr, 8, i)]
}

#Module for Application Load balancer, this will create External Load balancer and internal load balancer
module "ALB" {
  source             = "./modules/ALB"
  name               = "ACS-ext-alb"
  vpc_id             = module.VPC.vpc_id
  public-sg          = module.security.ALB-sg
  private-sg         = module.security.IALB-sg
  public-subnet-1       = module.VPC.public_subnets-1
  public-subnet-2       = module.VPC.public_subnets-2
  private-subnet-1      = module.VPC.private_subnets-1
  private-subnet-2      = module.VPC.private_subnets-2
  load_balancer_type = "application"
  ip_address_type    = "ipv4"
}

module "security" {
  source = "./modules/security"
  vpc_id = module.VPC.vpc_id
}


module "autoscaling" {
  source            = "./modules/autoscaling"
  ami-web           = var.ami
  ami-bastion       = var.ami
  ami-nginx         = var.ami
  desired_capacity  = 2
  min_size          = 2
  max_size          = 2
  web-sg            = [module.security.web-sg]
  bastion-sg        = [module.security.bastion-sg]
  nginx-sg          = [module.security.nginx-sg]
  wordpress-alb-tgt = module.ALB.wordpress-tgt
  nginx-alb-tgt     = module.ALB.nginx-tgt
  tooling-alb-tgt   = module.ALB.tooling-tgt
  instance_profile  = module.VPC.instance_profile
  public_subnets    = [module.VPC.public_subnets-1, module.VPC.public_subnets-2]
  private_subnets   = [module.VPC.private_subnets-1, module.VPC.private_subnets-2]
  keypair           = var.keypair

}

# Module for Elastic File system; this module will create elastic file system in the webservers availablity
# zone and allow traffic fro the webservers

module "EFS" {
  source       = "./modules/EFS"
  efs-subnet-1 = module.VPC.private_subnets-1
  efs-subnet-2 = module.VPC.private_subnets-2
  efs-sg       = [module.security.datalayer-sg]
  account_no   = var.account_no
}

# RDS module; this module will create the RDS instance in the private subnet

module "RDS" {
  source          = "./modules/RDS"
  master-password     = var.master-password
  master-username     = var.master-username
  db-sg           = [module.security.datalayer-sg]
  private_subnets = [module.VPC.private_subnets-3, module.VPC.private_subnets-4]
}

# The Module creates instances for jenkins, sonarqube abd jfrog
module "compute" {
  source          = "./modules/compute"
  ami-jenkins     = var.ami
  ami-sonar       = var.ami
  ami-jfrog       = var.ami
  subnets-compute = module.VPC.public_subnets-1
  sg-compute      = [module.security.ALB-sg]
  keypair         = var.keypair
}
```
- variables.tf - Here is where all the variables 
```
variable "region" {
  default = "us-east-1"
}

variable "vpc_cidr" {
  default = "172.16.0.0/16"
}

variable "enable_dns_support" {
  default = "true"
}

variable "enable_dns_hostnames" {
  default = "true"
}

variable "enable_classiclink" {
  default = "false"
}

variable "enable_classiclink_dns_support" {
  default = "false"
}

variable "preferred_number_of_public_subnets" {
  type        = number
  description = "Number of public subnets"
}

variable "preferred_number_of_private_subnets" {
  type        = number
  description = "Number of private subnets"
}

variable "name" {
  type        = string
  description = "ACS"
}

variable "tags" {
  description = "A mapping of tags assigned to all resources"
  type        = map(string)
  default     = {}
}

variable "ami" {
  type        = string
  description = "AMI ID for the launch template"
}

variable "keypair" {
  type        = string
  description = "Key pair for instances"
}

variable "account_no" {
  type        = number
  description = "account no"
}

variable "master-username" {
  type        = string
  description = "RDS admin username"
}

variable "master-password" {
  type        = string
  description = "RDs admin password"
}
```
- `terraform.tfvars` - All the varibles in the variable.tf of the root module is given a value here:
```
region = "us-east-1"

vpc_cidr = "172.16.0.0/16"

enable_dns_support = "true"

enable_dns_hostnames = "true"

enable_classiclink = "false"

enable_classiclink_dns_support = "false"

preferred_number_of_public_subnets = 2
  
preferred_number_of_private_subnets = 4

environment = "dev"

ami = "ami-09e67e426f25ce0d7"

keypair = "NvirgKP3"

master-password = "cynthiasproject"

master-username = "cynthia"

account_no = "148029617773"

tags = {
  Owner-Email = "adaora.okoduwa@gmail.com"
  Managed-By  = "Terraform"
  Billing-Account = "1234567890"
}
```
- providers.tf: paste the provider block in it:
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = var.region
}
```

### Backend on S3
Create an S3 bucket to store Terraform state file. Note: S3 bucket names must be unique unique within a region partition. S3 stores the state as a given key in a given bucket on Amazon S3. This backend also supports state locking and consistency checking via Dynamo DB
1. Create an S3 bucket in the main.tf file located in the root module and enable verionsing and encrption on the bucket with the following:
```
##creating bucket for s3 backend

resource "aws_s3_bucket" "terraform-state" {
  bucket        = "cynthiaspbl18"
  force_destroy = true
}
resource "aws_s3_bucket_versioning" "version" {
  bucket = aws_s3_bucket.terraform-state.id
  versioning_configuration {
    status = "Enabled"
  }
}
resource "aws_s3_bucket_server_side_encryption_configuration" "first" {
  bucket = aws_s3_bucket.terraform-state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```
2. Also in the same file create the DynamoDB table for state locking and consistency checking:
```
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}
```
3. Create `backend.tf` file in the root directory. Before we configure backend, Terraform expects that both S3 bucket and DynamoDB resources are already created before we configure the backend. So, let us run `terraform apply` to provision resources. 
4. Next create backend on S3 with the following code:
```
terraform {
   backend "s3" {
     bucket         = "cynthiaspbl18"
     key            = "global/s3/terraform.tfstate"
     region         = "us-east-1"
     dynamodb_table = "terraform-locks"
     encrypt        = true
   }
 }
 ```
 5. Re-initialize the backend with `terraform init`. Run terraform init and confirm you are happy to change the backend by typing `yes`
 6. Verify the changes, if you opened AWS now to see what happened you should be able to see the following:
- tfstatefile is now inside the S3 bucket
- DynamoDB table which we create has an entry which includes state file status
![pix5](https://user-images.githubusercontent.com/74002629/200835764-156977a8-b02b-446f-b2fc-0eeb3397cefe.PNG)
![pix6](https://user-images.githubusercontent.com/74002629/200835791-72c13d30-8b23-4451-93e5-6df6e83bb794.PNG)
![pix8](https://user-images.githubusercontent.com/74002629/200835887-2b8e1f6b-3549-4ab8-83a9-f2411217e830.PNG)
![pix9](https://user-images.githubusercontent.com/74002629/200835958-05387704-f520-42d3-b094-2f63bb4d5896.PNG)

7. To end the project, remember to destroy every resource you created. But before you destory make sure to migrate the backend back to your local machine by running `terraform init -migrate-state`
8. After maigrating the backend, run `terraform destroy` to destroy all the resources created.


