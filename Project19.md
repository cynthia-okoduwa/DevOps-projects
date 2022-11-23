## Automate Infrastructure With IaC using Terraform. Part 4 – Terraform Cloud

This is the concluding part of the 4 part project on Infrastructure as code using terrraform.
In the previous projects we built infrastructure for a      architecture on our local machines. I this project we would be building the same architecture but this
Terraform Cloud. Terraform Cloud is a managed service that provides you with Terraform CLI to provision infrastructure, either on demand or in response to various events.

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
 the prompt and add your newly created repository to the workspace.
6. Move on to "Configure settings", provide a description for your workspace and leave all the remaining settings as default, click "Create workspace"
7. Configure variables. Terraform Cloud supports two types of variables: Environment variables and Terraform variables. Either type can be marked as sensitive, 
to prevents them from being displayed in the Terraform Cloud web UI and makes them write-only. We will set two environment variables: **AWS_ACCESS_KEY_ID** and 
**AWS_SECRET_ACCESS_KEY**, set the values that you used in Project 16. These credentials will be used to privision your AWS infrastructure by Terraform Cloud.
After you have set these 2 environment variables – your Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources.
8. Now it is time to run our Terrafrom scripts, we would be using Packer to build our images in this project, and Ansible to configure the infrastructure, so for that 
we would be making changes to our our existing respository from Project 18. Add the following folders in your code structure:
- AMI: for building packer images
- Ansible: for Ansible scripts to configure the infrastucture
9. Install the following tools on your local machine:
- [Packer](https://developer.hashicorp.com/packer/tutorials/docker-get-started/get-started-install-cli)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

#### Create AMI using Packer

