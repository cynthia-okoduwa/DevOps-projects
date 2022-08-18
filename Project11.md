## ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

![Capture1](https://user-images.githubusercontent.com/74002629/185382955-28d67f00-8b19-4caa-8dd2-048cea6c0b74.PNG)

#### Task
1. Install and configure Ansible client to act as a Jump Server/Bastion Host
2. Create a simple Ansible playbook to automate servers configuration

### Install and configure Ansible client to act as a Jump Server/Bastion Host

An SSH jump server is a regular Linux server, accessible from the Internet, which is used as a gateway to access other Linux machines on a private network using the SSH protocol. Sometimes an SSH jump server is also called a “jump host” or a “bastion host”. The purpose of an SSH jump server is to be the only gateway for access to your infrastructure reducing the size of any potential attack surface.

#### Step 1 - INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE
1. Continuating from [project 9](https://github.com/cynthia-okoduwa/DevOps-projects/blob/main/Project9.md), update Name tag on your Jenkins EC2 Instance to **Jenkins-Ansible**. This server will be used to run playbooks.
2. In your GitHub account create a new repository and name it **ansible-config-mgt**.
3. In your **Jenkin-Ansible** server, instal **Ansible**
```
sudo apt update
sudo apt install ansible
```
4. Check your Ansible version by running `ansible --version`
5. Configure Jenkins build job to save your repository content every time you change it. See [project 9](https://github.com/cynthia-okoduwa/DevOps-projects/blob/main/Project9.md) for detailed steps
  - Create a new Freestyle project ansible in Jenkins and point it to your **ansible-config-mgt** repository.
  - Configure Webhook in GitHub and set webhook to trigger ansible build.
  ![pix1](https://user-images.githubusercontent.com/74002629/185372369-e33c094e-f075-4bdc-a4f3-e8dad525b60d.PNG)
  
  - Configure a Post-build job to save all (**) files. 
  - Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves 
    the files (build artifacts) in following folder `ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`
  ![pix2](https://user-images.githubusercontent.com/74002629/185372377-a6e7429c-e066-40f6-a098-961d3681b14f.PNG)
  ![pix6](https://user-images.githubusercontent.com/74002629/185372410-082abc5b-7212-4a42-bb20-532118c46458.PNG)
    
#### Step 2 – Prepare your development environment using Visual Studio Code
1. Install Visual Studio Code (VSC)- an Integrated development environment (IDE) or Source-code Editor. You can get it [here](https://code.visualstudio.com/download)
2. After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.
3. Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance: `git clone <ansible-config-mgt repo link>`

### Create a simple Ansible playbook to automate servers configuration

#### Step 3 - Begin Ansible development
1. In your **ansible-config-mgt** GitHub repository, create a new branch that will be used for development of a new feature.
2. Checkout the newly created feature branch to your local machine and start building your code and directory structure
3. Create a directory and name it **playbooks** – it will be used to store all your playbook files.
4. Create a directory and name it **inventory** – it will be used to keep your hosts organised.
5. Within the playbooks folder, create your first playbook, and name it **common.yml**
6. Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) **dev**, **staging**, **uat**, and **prod** respectively.

#### Step 4 – Set up an Ansible Inventory
An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since the intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

1. Save below inventory structure in the inventory/dev file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.
2. Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent: 
```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```
3. Confirm the key has been added with this command, you should see the name of your key: `ssh-add -l`
![pix8](https://user-images.githubusercontent.com/74002629/185372433-a4eb4ba5-d290-422b-91e6-8a5260e0dad5.PNG)

5. Now, ssh into your Jenkins-Ansible server using ssh-agent: `ssh -A ubuntu@public-ip`
6. Also notice, that your ubuntu user is ubuntu and user for RHEL-based servers is ec2-user.
7. Update your inventory/dev.yml file with this snippet of code:
```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```
![pix11](https://user-images.githubusercontent.com/74002629/185373588-0cb4a21a-d0a6-4bb3-9c21-475bc402011f.PNG)

#### Step 5 – Create a Common Playbook
Now we give Ansible the instructions on what you needs to be performed on all servers listed in **inventory/dev**. In **common.yml** playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.
1. Update your playbooks/common.yml file with following code:
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```
![pix12](https://user-images.githubusercontent.com/74002629/185373600-c9815226-51e1-4e1a-ac92-b17b2e3713ea.PNG)

2. This playbook is divided into two parts, each of them is intended to perform the same task: install **wireshark utility** (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses **root** user to perform this task and respective package manager: **yum** for RHEL 8 and **apt** for Ubuntu.
3. For a better understanding of Ansible playbooks – [watch this video](https://www.youtube.com/watch?v=ZAdJ7CdN7DY) and read [this article](https://www.redhat.com/en/topics/automation/what-is-an-ansible-playbook) from Redhat.

#### Step 6 – Update GIT with the latest code
1. Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.
2. Commit your code into GitHub: use git commands to **add**, **commit** and **push** your branch to GitHub.
```
git status
git add <selected files>
git commit -m "commit message"
```
4. Create a Pull request (PR)
![pix14](https://user-images.githubusercontent.com/74002629/185374143-0881f820-48ac-4ff6-bafe-4a8d9c180341.PNG)

3. Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to **/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/** directory on Jenkins-Ansible server.

![pix17](https://user-images.githubusercontent.com/74002629/185374194-509b7ab2-0007-46ac-8e78-836a249ec73c.PNG)

#### Step 7 – Run Ansible test
1. Now, it is time to execute ansible-playbook command and verify if your playbook actually works: `cd ansible-config-mgt`
2. Run ansible-playbook command: `ansible-playbook -i inventory/dev.yml playbooks/common.yml`
![pix20](https://user-images.githubusercontent.com/74002629/185374713-40418adb-3758-4b45-823e-a2825607d3f5.PNG)

4. If your command ran successfully, go to each of the servers and check if wireshark has been installed by running `which wireshark` or `wireshark --version`
![pix22](https://user-images.githubusercontent.com/74002629/185374839-0f2a05ba-78f7-44c5-abc6-d72c84c258de.PNG)
![pix23](https://user-images.githubusercontent.com/74002629/185374858-d24eacde-dbf0-46f9-a3e5-72ede5f3b0cd.PNG)
