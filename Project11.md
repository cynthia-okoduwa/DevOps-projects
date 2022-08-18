# ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

#### Task
1. Install and configure Ansible client to act as a Jump Server/Bastion Host
2. Create a simple Ansible playbook to automate servers configuration

### Install and configure Ansible client to act as a Jump Server/Bastion Host

#### Step 1 - INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE
1. Update Name tag on your Jenkins EC2 Instance to **Jenkins-Ansible**. This server will be used to run playbooks.
2. In your GitHub account create a new repository and name it **ansible-config-mgt**.
3. In your **Jenkin-Ansible** server, instal **Ansible**
```
sudo apt update
sudo apt install ansible
```
4. Check your Ansible version by running `ansible --version`
5. Configure Jenkins build job to save your repository content every time you change it. (See project 9 for detailed steps)
  - Create a new Freestyle project ansible in Jenkins and point it to your **ansible-config-mgt** repository.
  - Configure Webhook in GitHub and set webhook to trigger ansible build.
  - Configure a Post-build job to save all (**) files. 
  - Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves 
    the files (build artifacts) in following folder `ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
    
Step 2 – Prepare your development environment using Visual Studio Code
1. Install Visual Studio Code (VSC)- an Integrated development environment (IDE) or Source-code Editor. You can get it [here](https://code.visualstudio.com/download)

After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance

git clone <ansible-config-mgt repo link>
