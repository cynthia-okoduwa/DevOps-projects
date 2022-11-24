## Automate Infrastructure With IaC using Terraform. Part 4 – Terraform Cloud
![Capture](https://user-images.githubusercontent.com/74002629/203817812-62cf36bc-623c-4182-b139-da2580de548b.PNG)

This is the concluding part of the 4-part project on Infrastructure as Code using terraform. In the previous projects we built infrastructure for our architecture on our local machines. In this project we would be building the same architecture but this time with Terraform Cloud. Terraform Cloud is a managed service that provides you with Terraform CLI to provision infrastructure, either on demand or in response to various events.

#### Migrate your .tf codes to Terraform Cloud
##### Steps
1. Create a new account with this [link](https://app.terraform.io/signup/account), then verify your email and you are ready.
2. Create an organization, select "Start from scratch", choose a name for your organization and create it.
3. Next, configure a workspace. There are 3 options to configure your workspace: 
- Version Control workflow: This is used to integrate your version control systems like GitHub, Gitlab, etc. When you publish a new version to the default branch, 
it would trigger an automatic plan and also apply to target branch if you set it to do so. It is the most commonly used.
-  CLI-driven workflow: With this you can run terraform commands from your local CLI but you are integrated with terraform cloud using some special sign in 
and settings and it will run your commands in the cloud and make use of that state.
- API-driven workflow: This allows you work with APIs

4. We will use version control workflow as the most common and recommended way to run Terraform commands triggered from our git repository. Create a new repository
 in your GitHub and call it `terraform-cloud`, push your Terraform codes developed in the previous projects to the repository.
5. Choose Version Control Workflow and you will be promped to connect to your Version Control system account to your workspace (choose which ever suits you). 
You will be required to register a new OAuth Application – follow the prompt and connect your newly created repository to the workspace.
6. Move on to "Configure settings", provide a description for your workspace and leave all the remaining settings as default, click "Create workspace"
7. Configure variables. Terraform Cloud supports two types of variables: Environment variables and Terraform variables. Either type can be marked as sensitive, 
to prevents them from being displayed in the Terraform Cloud web UI and makes them write-only. We will set two environment variables: **AWS_ACCESS_KEY_ID** and 
**AWS_SECRET_ACCESS_KEY**, set the values that you used in Project 16. These credentials will be used to provision your AWS infrastructure by Terraform Cloud.
For the Terraform variables instead entering each variable we have created in our `variables.tfvars` file here, simply change the file name from `variables.tfvars` to `variables.auto.tfvars` in the terraform-cloud directory structure. Terraform cloud will automatically pick the vaules in the file directly.
After you have set the 2 variables – your Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources.
8. Now it is time to run our Terrafrom scripts, we would be using Packer to build our images in this project, and Ansible to configure the infrastructure, so for that 
we would be making changes to our our existing respository from [Project 18](https://github.com/cynthia-okoduwa/DevOps-projects/blob/main/Project18.md). Add the following folders in your code structure:
- AMI: for building packer images
- Ansible: for Ansible scripts to configure the infrastucture
9. Install the following tools on your local machine:
- [Packer](https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli) To create custom images that are immutable and prodduction ready.
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) for your server configuration.

#### Create AMI using Packer
1. Terraform-cloud file structure create a new folder and name it `AMI`, the move all the .sh files from project 19 into it.(bastion.sh, ubuntu.sh, web.sh, nginx.sh)
2. Create 4 new files in the folder and name them as `bastion.pkr.hcl`, `nginx.pkr.hcl`, `web.pkr.hcl` and `ubuntu.pkr.hcl` respectively.
3. Inside the `bastion.pkr.hcl` paste the following user data :
```
variable "region" {
  type    = string
  default = "us-east-2"
}

locals {
  timestamp = regex_replace(timestamp(), "[- TZ:]", "")
}


# source blocks are generated from your builders; a source can be referenced in
# build blocks. A build block runs provisioners and post-processors on a
# source.
source "amazon-ebs" "terraform-bastion-prj-19" {
  ami_name      = "terraform-bastion-prj-19-${local.timestamp}"
  instance_type = "t2.micro"
  region        = var.region
  source_ami_filter {
    filters = {
      name                = "RHEL-SAP-8.2.0_HVM-20211007-x86_64-0-Hourly2-GP2"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["309956199498"]
  }
  ssh_username = "ec2-user"
  tag {
    key   = "Name"
    value = "terraform-bastion-prj-19"
  }
}

# a build block invokes sources and runs provisioning steps on them.
build {
  sources = ["source.amazon-ebs.terraform-bastion-prj-19"]

  provisioner "shell" {
    script = "bastion.sh"
  }
}
```
4. Inside `nginx.pkr.hcl` paste:
```
variable "region" {
  type    = string
  default = "us-east-2"
}

locals { timestamp = regex_replace(timestamp(), "[- TZ:]", "") }


# source blocks are generated from your builders; a source can be referenced in
# build blocks. A build block runs provisioners and post-processors on a
# source.
source "amazon-ebs" "terraform-nginx-prj-19" {
  ami_name      = "terraform-nginx-prj-19-${local.timestamp}"
  instance_type = "t2.micro"
  region        = var.region
  source_ami_filter {
    filters = {
      name                = "RHEL-SAP-8.2.0_HVM-20211007-x86_64-0-Hourly2-GP2"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["309956199498"]
  }
  ssh_username = "ec2-user"
  tag {
    key   = "Name"
    value = "terraform-nginx-prj-19"
  }
}


# a build block invokes sources and runs provisioning steps on them.
build {
  sources = ["source.amazon-ebs.terraform-nginx-prj-19"]

  provisioner "shell" {
    script = "nginx.sh"
  }
}
```
5. Inside `ubuntu.pkr.hcl` paste:
```
variable "region" {
  type    = string
  default = "us-east-2"
}

locals { timestamp = regex_replace(timestamp(), "[- TZ:]", "") }


# source blocks are generated from your builders; a source can be referenced in
# build blocks. A build block runs provisioners and post-processors on a
# source.
source "amazon-ebs" "terraform-ubuntu-prj-19" {
  ami_name      = "terraform-ubuntu-prj-19-${local.timestamp}"
  instance_type = "t2.micro"
  region        = var.region
  source_ami_filter {
    filters = {
      name                = "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["099720109477"]
  }
  ssh_username = "ubuntu"
  tag {
    key   = "Name"
    value = "terraform-ubuntu-prj-19"
  }
}


# a build block invokes sources and runs provisioning steps on them.
build {
  sources = ["source.amazon-ebs.terraform-ubuntu-prj-19"]

  provisioner "shell" {
    script = "ubuntu.sh"
  }
}
```
6. For web.pkr.hcl, paste:
```
variable "region" {
  type    = string
  default = "us-east-2"
}

locals { timestamp = regex_replace(timestamp(), "[- TZ:]", "") }


# source blocks are generated from your builders; a source can be referenced in
# build blocks. A build block runs provisioners and post-processors on a
# source.
source "amazon-ebs" "terraform-web-prj-19" {
  ami_name      = "terraform-web-prj-19-${local.timestamp}"
  instance_type = "t2.micro"
  region        = var.region
  source_ami_filter {
    filters = {
      name                = "RHEL-SAP-8.2.0_HVM-20211007-x86_64-0-Hourly2-GP2"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["309956199498"]
  }
  ssh_username = "ec2-user"
  tag {
    key   = "Name"
    value = "terraform-web-prj-19"
  }
}


# a build block invokes sources and runs provisioning steps on them.
build {
  sources = ["source.amazon-ebs.terraform-web-prj-19"]

  provisioner "shell" {
    script = "web.sh"
  }
}
```
7. These Packer configurations will make use of the user data provided in their respective .sh files.
8. Next in your terminal, navigate to the terraform-cloud directory and begin building your AMI. Type `packer build <name of packer file>` to build each packer AMI
9. Once build is complete, you should see your Web, Bastion, Nginx and Ubuntu AMIs in your AWS console.
![AMIs](https://user-images.githubusercontent.com/74002629/203560607-aba1b0bb-e8ac-461e-8d11-f490fe18afe2.PNG)
10. Copy the AMI ID of each AMI created and update the terraform.auto.tfvars file with the newly created AMIs. Note: the Ubuntu AMI will be used for Sonarqube.
![AMI ID](https://user-images.githubusercontent.com/74002629/203724846-2ec829c7-3795-4366-9d00-309b3e4c988f.PNG)
11. Push all your changes to the `terraform-cloud` repository.
12. Next in the Terraform cloud UI, run your first plan, if all goes well, run Apply. You will see your AWS resources being created.
![Pix2](https://user-images.githubusercontent.com/74002629/203728085-305eb60b-fa6b-433b-8f46-c88377f42ad3.PNG)
13. You have successfully created your resources using Terraform cloud. When you go into your AWS console you should see all the resources you have created, however our instances in the target group have failed health checks, beacause we have not configured the instances. Let's fix that.
14. In our terraform code, remove the instances as listeners and attachment to auto scaling groups. Comment out the nginx, loadbalancer and tooling listeners in the ALB module of your terraform code. and also comment out the attachment of autoscaling groups to the load balancers in the Autoscaling module of your terraform code. We are doing this to prevent issues till we have run our configurations, then we can reapply them.
15. Push your changes and Terraform cloud with Plan and Apply automatically

#### Update Ansible script with values from Terraform output.
1. Next we would be updating the Ansible script with values from terraform output. 
2. SSH into your Bastion server using SSH agent and clone down your repository.
3. Ansible will need to have access to your AWS accout to pull down the IP addresses of your instances, we will need to give Ansible access. Enter `aws configure` in your ansible directory, then follow the prompt and provide your secret key and access key to give Ansible access.
4. Ensure Ansible can pull down the required IPs by runing: `ansible-inventory -i invetory/aws.ec2.yml --graph` 
5. Update the nginx role with DNS name for the laodbalancer. Go into your loadbalancer in your AWS console and copy the DNS name for the loadbalance and paste it in the nginx role.
6. Next update the RDS end-point in the tooling/tasks/setup-db.yml  and wordpress/tasks/setup-db.yml files (You get this from your AWS console from the RDS you provisioned) Update it for the database and tooling credentials.
7. Also update your username name and password to correspond to what you have in your terraform.auto.tfvars
8. Next, update the access points of your filesystem for both the tooling and the wordpress site. it is located in `Ansible/roles/tooling/tasks/main.yml` and `Ansible/roles/wordpress/tasks/main.yml` respectively.
9. In the Ansible folder, create an ansible.cfg file and specify your roles path to allow Ansible find the roles when it runs, then run `export ANSIBLE_CONFIG=<path to your ansible.cfg file>` in the terminal to tell ansible where to find the roles
10. Now run Ansible playbook against your environment. `ansible-playbook -i invetory/aws.ec2.yml playbooks/site.yml` If all goes well, you should see your playbook running.
11. Now we get into your Bastion server via SSH agent. Type `ssh -A ec2-user@<Bastion-Public-IP>` and from inside the bastion we can get into the other servers in the architecture.
12. To get into the other servers from the Bastion using SSH agent, enter the command `ssh ec2-user@<Private-IP-of-target-server>
13. Once inside your Nginx server run `sudo systemctl status nginx` to verify the server is running, the `sudo vi /etc/nginx/nginx.conf` to verify everything was configured correctly.
14. SSH into your tooling and wordpress servers respectively and run `df -h` to verify that the filesystem was successfully mounted and `sudo systemctl status httpd` to verif Apache is running.
15. in each server run change directory to `cd /var/www/html/` to see the health status of your servers, and `curl localhost` to see the webserver running in your local machine.





