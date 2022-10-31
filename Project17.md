## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 2
In continuation to [Project16](https://github.com/cynthia-okoduwa/DevOps-projects/blob/main/project16.md), in this project we continue to create the other resources in our architecture. The resources we will be creating includes:
1. For Networking, in addition to the VPC and 2 public subnets created in project 16, we will also create:
- 4 private subnets
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

#### Create Private subnets
1. Let's modify our code used in creating the public subnets and create the private subnets:
```
# Create private subnets
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
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

#### Internet Gateways & format() function
1. Create a new file and name it `internet_gateway.tf` then creat an Internet Gateway in file with the following code:
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
#### NAT Gateway
1. Create a NAT Gateways and an Elastic IP (EIP) addresse and allocate the EIP to the NAT gateway. Create the NAT Gateway in a new file called `natgateway.tf` and use the following code snippet to create:
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
#### Route tables
1. Next, create a file called `route_tables.tf``` and inside it create routes for both public and private subnets, and a route for te the internet gateway, create the below resources. Ensure they are properly tagged.
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
4. Now if you run `terraform plan` and `terraform apply` you will have the following resources in your AWS infrastructure in multi-az set up:
- Our vpc
- 2 Public subnets
- 4 Private subnets
- 1 Internet Gateway
- 1 NAT Gateway
- 1 EIP
- 2 Route tables

#### AWS IDENTITY AND ACCESS MANAGEMENT

Our EC2 instances that we will be creating later will need to have access to some resources in our infrastucture, we want to pass an IAM role them as required by the architecture.

1. Create **AssumeRole**: Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically, you use AssumeRole within your account or for cross-account access. Add the following code to a new file named roles.tf and tag appropriately.
```
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
```
In this code we are creating AssumeRole with AssumeRole policy. It grants to an entity, in our case it is an EC2, permissions to assume the role.

2. Create IAM policy for this role: This is where we need to define a required policy (i.e., permissions) according to our requirements. For example, allowing an IAM role to perform action describe applied to EC2 instances:
```
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
```
3. Attach the Policy to the IAM Role: here we will be attaching the policy we created to the role we created in the first step.
```
    resource "aws_iam_role_policy_attachment" "test-attach" {
        role       = aws_iam_role.ec2_instance_role.name
        policy_arn = aws_iam_policy.policy.arn
    }
```
4. Create an [AWS Instance Profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html) and interpolate the IAM Role
```
    resource "aws_iam_instance_profile" "ip" {
        name = "aws_instance_profile_test"
        role =  aws_iam_role.ec2_instance_role.name
    }
```
#### CREATE SECURITY GROUPS
We are creating security groups and security group rules for the resources below. Click the links to learn more about creating [security groups](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) and [security groups rules](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group_rule)
- External Application loadbalancers
- Internal Application loadbalancers
- Webservers
- Data layer
- Bastion host
- Nginx reverse proxy
1. These security groups will be created in a single new file named `security.tf` , then we are going to refrence this security group within each resources that needs it. Create security groups with the following code snippet:
```
# security group for alb, to allow acess from any where for HTTP and HTTPS traffic
resource "aws_security_group" "ext-alb-sg" {
  name        = "ext-alb-sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow TLS inbound traffic"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

 tags = merge(
    var.tags,
    {
      Name = "ext-alb-sg"
    },
  )

}


# security group for bastion, to allow access into the bastion host from you IP
resource "aws_security_group" "bastion_sg" {
  name        = "bastion_sg"
  vpc_id = aws_vpc.main.id
  description = "Allow incoming HTTP connections."

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

   tags = merge(
    var.tags,
    {
      Name = "Bastion-SG"
    },
  )
}



#security group for nginx reverse proxy, to allow access only from the extaernal load balancer and bastion instance
resource "aws_security_group" "nginx-sg" {
  name   = "nginx-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

   tags = merge(
    var.tags,
    {
      Name = "nginx-SG"
    },
  )
}

resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ext-alb-sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}


# security group for ialb, to have acces only from nginx reverser proxy server
resource "aws_security_group" "int-alb-sg" {
  name   = "my-alb-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "int-alb-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-ialb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.nginx-sg.id
  security_group_id        = aws_security_group.int-alb-sg.id
}

 
# security group for webservers, to have access only from the internal load balancer and bastion instance
resource "aws_security_group" "webserver-sg" {
  name   = "webserver-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "webserver-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.int-alb-sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}


# security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port
resource "aws_security_group" "datalayer-sg" {
  name   = "datalayer-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

 tags = merge(
    var.tags,
    {
      Name = "datalayer-sg"
    },
  )
}

resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}
```
#### CREATE CERTIFICATE FROM AMAZON CERIFICATE MANAGER
You would require a domain name for this part of the project. Be sure to check out the terraform documentation for [AWS certificate manager](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/acm_certificate)
1. Create `cert.tf file` and add the following code snippets to it.
NOTE: Read Through to change the domain name to your own domain name and every other name that needs to be changed.
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
#### Create an external (Internet facing) and an Internal Application Load-Balancer (ALB)
1. We will create an ALB that receives traffic from the internet and distributes it between the nginx reverse proxies and an internal load-balancers that receives traffic from the ngix reverse proxies and distributes to the web-servers. First, we will create the ALB, then the target group and lastly, we will create the listener rule. Create a file called `alb.tf` and paste the following code snippet:
```
# External loadbalancer
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

# ---Internal Load Balancers for webservers----

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
  vpc_id      = aws_vpc.main.id
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
To learn more about the argument needed for each resource, click the following links [ALB](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb), [Target group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_target_group), [ALB-listener](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/lb_listener)

2. Add the following outputs to `output.tf` to print them on screen
```
output "alb_dns_name" {
  value = aws_lb.ext-alb.dns_name
}

output "alb_target_group_arn" {
  value = aws_lb_target_group.nginx-tgt.arn
}
```
#### Create Autoscaling groups
Next, we will be creating Auto Scaling Group (ASG) to allow our architecture scale the EC2s in and out depending on the amount of traffic coming into our infrastructure. Before configuring an ASG, we need to create the launch template and the AMI the AGS needs. 
Based on our architecture we need to create Auto-scaling groups for bastion, nginx, wordpress and tooling, so we will create two files; `asg-bastion-nginx.tf` will contain Launch template and austo-scaling group for Bastion and Nginx, then `asg-wordpress-tooling.tf` will contain Launch template and austo-scaling group for wordpress and tooling. Here are some useful Terraform documentation, to understand the arguements needed for each resources:
[SNS-topic](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/sns_topic),
[SNS-notification](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_notification),
[Austoscaling](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/autoscaling_group),
[Launch-template](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/launch_template).

1. Create `asg-bastion-nginx.tf` and paste all the code snippet below;
```
# creating sns topic for all the auto scaling groups
resource "aws_sns_topic" "cynthia-sns" {
name = "Default_CloudWatch_Alarms_Topic"
}
```
2. Create notification for all the auto scaling groups:
```
resource "aws_autoscaling_notification" "cynthia_notifications" {
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

  topic_arn = aws_sns_topic.cynthia-sns.arn
}
```
3. Create Launch template and Atuo scaling groups for bastion
```
# launch template for bastion

resource "random_shuffle" "az_list" {
  input        = data.aws_availability_zones.available.names
}

resource "aws_launch_template" "bastion-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.bastion_sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
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

# ---- Autoscaling for bastion  hosts

resource "aws_autoscaling_group" "bastion-asg" {
  name                      = "bastion-asg"
  max_size                  = 2
  min_size                  = 2
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 2

  vpc_zone_identifier = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  launch_template {
    id      = aws_launch_template.bastion-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "bastion-launch-template"
    propagate_at_launch = true
  }

}
```
4. Inside the same file create Launch template and Auto scaling group for nginx server:
```
# launch template for nginx

resource "aws_launch_template" "nginx-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.nginx-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name =  var.keypair

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

# ------ Autoscslaling group for reverse proxy nginx ---------

resource "aws_autoscaling_group" "nginx-asg" {
  name                      = "nginx-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  launch_template {
    id      = aws_launch_template.nginx-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "nginx-launch-template"
    propagate_at_launch = true
  }

}

# attaching autoscaling group of nginx to external load balancer
resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
  autoscaling_group_name = aws_autoscaling_group.nginx-asg.id
  alb_target_group_arn   = aws_lb_target_group.nginx-tgt.arn
}
```
5. Create a new file and name it `asg-wordpress-tooling.tf`. This is where the Launch template for the Wordpress and Tooling site would be created.
6. Create Launch template and Autoscaling group for Wordpress and attach to internal loadbalancer:
```
# launch template for wordpress

resource "aws_launch_template" "wordpress-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.webserver-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
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

# ---- Autoscaling for wordpress application

resource "aws_autoscaling_group" "wordpress-asg" {
  name                      = "wordpress-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1
  vpc_zone_identifier = [

    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  launch_template {
    id      = aws_launch_template.wordpress-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "wordpress-asg"
    propagate_at_launch = true
  }
}

# attaching autoscaling group of  wordpress application to internal loadbalancer
resource "aws_autoscaling_attachment" "asg_attachment_wordpress" {
  autoscaling_group_name = aws_autoscaling_group.wordpress-asg.id
  alb_target_group_arn   = aws_lb_target_group.wordpress-tgt.arn
}
```
7. Create Launch template and Autoscaling group for Wordpress and attach to internal loadbalancer:
```
# launch template for toooling
resource "aws_launch_template" "tooling-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.webserver-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
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

# ---- Autoscaling for tooling -----

resource "aws_autoscaling_group" "tooling-asg" {
  name                      = "tooling-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [

    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  launch_template {
    id      = aws_launch_template.tooling-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "tooling-launch-template"
    propagate_at_launch = true
  }
}

# attaching autoscaling group of  tooling application to internal loadbalancer
resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
  autoscaling_group_name = aws_autoscaling_group.tooling-asg.id
  alb_target_group_arn   = aws_lb_target_group.tooling-tgt.arn
}

```

### STORAGE AND DATABASE
The final group of resources to create are the Elastic File System (EFS) and Relational Database Service (RDS).

1. Create Elastic File System (EFS): to follow best practice in using EFS for file sharing, we need a KMS key. AWS Key Management Service (KMS) makes it easy for you to create and manage cryptographic keys and control their use across a wide range of AWS services and in your applications.
To create an EFS you need to create a KMS key.

2. Create a new file and name it `efs.tf` and paste the following in it. This would create the KMS key, EFS and the mount targets for the the EFS:
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
  subnet_id       = aws_subnet.private[0].id
  security_groups = [aws_security_group.datalayer-sg.id]
}

# set second mount target for the EFS 
resource "aws_efs_mount_target" "subnet-2" {
  file_system_id  = aws_efs_file_system.ACS-efs.id
  subnet_id       = aws_subnet.private[1].id
  security_groups = [aws_security_group.datalayer-sg.id]
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
3. Next Create MySQL RDS: create a new file named `rds.tf` and paste the following code snippet to create:
```
# This section will create the subnet group for the RDS  instance using the private subnet
resource "aws_db_subnet_group" "ACS-rds" {
  name       = "acs-rds"
  subnet_ids = [aws_subnet.private[2].id, aws_subnet.private[3].id]

 tags = merge(
    var.tags,
    {
      Name = "ACS-rds"
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
  name                   = "cynthiadb"
  username               = var.master-username
  password               = var.master-password
  parameter_group_name   = "default.mysql5.7"
  db_subnet_group_name   = aws_db_subnet_group.ACS-rds.name
  skip_final_snapshot    = true
  vpc_security_group_ids = [aws_security_group.datalayer-sg.id]
  multi_az               = "true"
}
```
### VARIABLES.TF AND TERRAFORM.TFVARS
So far, we have created all the elements in our infrastructure, in the process of creating our resources we referenced variables that we have not yet declared. So if we try to apply this it would fail. This is where the variables.tf and terraform.tfvars files come it. We would use the variables.tf file to declare all the variables that we referenced while creating our resources and then use the terraform.tfvars file to give value to these variables.
1. Create a new file and name it `variables.tf` then declare all the variables you had earlier referenced in your resources. You code should look like this:
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
  type = number
  description = "Number of public subnets"
}

variable "preferred_number_of_private_subnets" {
  type = number
  description = "Number of private subnets"
}

variable "name" {
    type = string
    description = "ACS"
}

variable "tags" {
    description = "A mapping of tags assigned to all resources"
    type = map(string)
    default = {}
}

variable "ami" {
  type = string
  description = "AMI ID for the launch template"
}

variable "keypair" {
  type = string
  description = "Key pair for instances"
}

variable "account_no" {
  type = number
  description = "account no"
}

variable "master-username" {
  type = string
  description = "RDS admin username"
}

variable "master-password" {
  type = string
  description = "RDs admin password"
}
```
3. Then create another file and name it `terraform.tfvars` and give your declared variables value, like this:
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
  type = number
  description = "Number of public subnets"
}

variable "preferred_number_of_private_subnets" {
  type = number
  description = "Number of private subnets"
}

variable "name" {
    type = string
    description = "ACS"
}

variable "tags" {
    description = "A mapping of tags assigned to all resources"
    type = map(string)
    default = {}
}

variable "ami" {
  type = string
  description = "AMI ID for the launch template"
}

variable "keypair" {
  type = string
  description = "Key pair for instances"
}

variable "account_no" {
  type = number
  description = "account no"
}

variable "master-username" {
  type = string
  description = "RDS admin username"
}

variable "master-password" {
  type = string
  description = "RDs admin password"
}
```
At this point, our infrastructure elements are ready to be deployed automatically, but before we plan and apply the code we need to take note of two things;
- Our list of files is long and confusing but not bad for a start, we are going to fix this using the concepts of Modules.
- Secondly, our application won't work because in the shell script that was passed into the launch some endpoints like the RDs and EFS point is needed that have not been created yet. we would use  Ansible to fix this.
4. Finally, plan and apply your Terraform codes, explore the resources in AWS console and make sure you destroy them right away to avoid massive costs.
![pix15](https://user-images.githubusercontent.com/74002629/199027421-dde0d5b0-82c1-4412-a5c0-89a8ac0e5c70.PNG)
![pix4a](https://user-images.githubusercontent.com/74002629/199027016-33e0adb3-4e8b-4f64-aaac-436c897552cf.PNG)
![pix4b](https://user-images.githubusercontent.com/74002629/199027039-2c19d46c-9c1c-4d70-b8c5-d5b759664a61.PNG)
![pix4c](https://user-images.githubusercontent.com/74002629/199027061-f372a0b4-507d-42c7-a8c1-f2e553af1880.PNG)
![pix4d](https://user-images.githubusercontent.com/74002629/199027103-45e49d37-5cf1-4e65-bb2d-4038cfbfcd28.PNG)
![pix5](https://user-images.githubusercontent.com/74002629/199027119-b7c50219-f06b-4ad6-8820-e0343fb6d618.PNG)
![pix6](https://user-images.githubusercontent.com/74002629/199027129-8c28a96d-1e36-4b38-82f7-bfe74f8cd6b7.PNG)
![pix7](https://user-images.githubusercontent.com/74002629/199027144-86a4be90-2def-46e9-8f00-6ed80da571c2.PNG)
![pix11](https://user-images.githubusercontent.com/74002629/199027258-ec2710cd-0b64-4dad-8303-4c955ad0e785.PNG)
![pix13](https://user-images.githubusercontent.com/74002629/199027314-983e672b-95c6-4b70-a621-21dba46148a8.PNG)
![pix14](https://user-images.githubusercontent.com/74002629/199027371-5a9fd2ae-88bb-492f-ba27-d6e0b01cc3c8.PNG)

