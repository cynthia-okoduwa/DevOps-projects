## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

![Capture](https://user-images.githubusercontent.com/74002629/187196466-07bfe88d-4571-43ba-9d7e-fa757395432d.PNG)

### Task
1. Refactor your Ansible code, create assignments, and use the imports functionality.


### Code Refactoring
Refactoring is a general term in computer programming. It means making changes to the source code without changing expected 
behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, 
reduce complexity, add proper comments without affecting the logic.

### Step 1 – Jenkins job enhancement
Every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. 
Besides, it consumes space on Jenkins server with each subsequent change. I will enhance it by introducing a new Jenkins project/job – **Copy Artifact** plugin
will be required for this. 

1. Inside the  **Jenkins-Ansible** server and create a new directory called **ansible-config-artifact** – where we will store all artifacts after each build.
`sudo mkdir /home/ubuntu/ansible-config-artifact`
2. Change permissions to this directory, so Jenkins could save files there – `chmod -R 0777 /home/ubuntu/ansible-config-artifact`
3. Go to Jenkins web console -> **Manage Jenkins** -> **Manage Plugins** -> on **Available** tab search for **Copy Artifact** and install this plugin without 
restarting Jenkins.
4. Create a new **Freestyle project** (see [Project 9](https://github.com/cynthia-okoduwa/DevOps-projects/blob/main/Project9.md)) and name it 
**save_artifacts**.
5. This project will be triggered by completion of the existing ansible project. Configure it accordingly:
  - In **General** tab, enter your desired no for the **Max # of build to keep** (In my case 2)
  - Under Build Triggers, Enter **ansible.**
 ![Capture1](https://user-images.githubusercontent.com/74002629/187196502-3cddcd1e-5839-4930-93cc-67d43016a83f.PNG)

6. The main idea for save_artifacts project is to save artifacts into `/home/ubuntu/ansible-config-artifact directory`. To achieve this, create a build 
step and choose `Copy artifacts from other project`, specify **ansible** as a source project and `/home/ubuntu/ansible-config-artifact` as a target directory.

  ![Capture2](https://user-images.githubusercontent.com/74002629/187196539-b70887b6-587f-4e85-8572-0f60e094e028.PNG)
7. Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master branch).
If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated 
with every commit to your master branch.
  ![Capture3](https://user-images.githubusercontent.com/74002629/187196549-ccd85653-08eb-4335-ac99-5bd1436dcf7c.PNG)

### Step 2 – Refactor Ansible code by importing other playbooks into site.yml
In Project 11 I wrote all tasks in a single playbook **common.yml**, a simple set of instructions for only 2 types of OS, but imagine there are many more tasks 
and I need to apply this playbook to other servers with different requirements. In this case, I will have to read through the whole playbook to check if all tasks 
written there are applicable and is there anything that you need to add for certain server/OS families. Very fast it will become a tedious exercise and my playbook 
will become messy.
Breaking up tasks into different files is an excellent way to organize complex sets of tasks and optimize your Playbooks.

Let see code re-use in action by importing other playbooks.
1. Before refactoring the codes, ensure you have pulled down the latest code from **master (main)** branch, and create a new branch, name it **refactor** in VScode.
2. Within playbooks folder, create a new file and name it **site.yml** – This file will now be considered as an entry point into the entire infrastructure 
configuration. In other words, site.yml will become a parent to all other playbooks that will be developed including common.yml that you created previously.
3. Create a new folder in root of the repository and name it **static-assignments**. The static-assignments folder is where all other children playbooks will be 
stored. This is merely for easy organization of your work. It is not an Ansible specific concept.
4. Move **common.yml** file into the newly created **static-assignments** folder.
![pix2](https://user-images.githubusercontent.com/74002629/187199132-243aa796-c6f0-4697-97d8-caa4aa53861a.PNG)

6. Inside **site.ym**l file, import **common.yml** playbook:
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
7. Run **ansible-playbook** command against the **dev** environment. Since you need to apply some tasks to your dev servers and wireshark is already 
installed – you can go ahead and create another playbook under static-assignments and name it common-del.yml. In this playbook, configure deletion of wireshark utility.
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```
8. update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers:
```
cd /home/ubuntu/ansible-config-mgt/
ansible-playbook -i inventory/dev.yml playbooks/site.yaml
```
![pix4](https://user-images.githubusercontent.com/74002629/187199159-819a2a3a-1c5e-49f7-91da-22e346baaa07.PNG)

9. Confirm that wireshark is deleted on all the servers by running `wireshark --version`

![pix6](https://user-images.githubusercontent.com/74002629/187199257-b770119d-ff7b-4390-b461-6ffe17a16c33.PNG)
![pix7](https://user-images.githubusercontent.com/74002629/187199276-e0b5e4e0-3a08-4466-8cc1-48b3ca2c9ce7.PNG)
### Step 3 – Configure UAT Webservers with a role ‘Webserver’
1. Launch 2 fresh EC2 instances using RHEL 8 image, name them accordingly – Web1-UAT and Web2-UAT.
2. To create a role, you must create a directory called **roles/**, relative to the playbook file or in **/etc/ansible/ directory**.
3. The entire folder structure should look like below:
```
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
```
4. Update your inventory **ansible-config-mgt/inventory/uat.yml** file with IP addresses of your 2 UAT Web servers.
```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
```

5. Use ssh-agent to ssh into the Jenkins-Ansible instance.
6. In **/etc/ansible/ansible.cfg** file uncomment **roles_path** string and provide a full path to your roles directory, so Ansible knows where to find configured roles.
`roles_path    = /home/ubuntu/ansible-config-mgt/roles`

![step3pix4](https://user-images.githubusercontent.com/74002629/187199413-51e7f0e2-9674-4f83-9e82-0148c05fd505.PNG)

7. Time to add some logic to the webserver role. Go into **tasks** directory, and within the **main.yml** file, write configuration tasks to do the following:
  - Install and configure Apache (httpd service)
  - Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
  - Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
  - Make sure httpd service is started
8. Your **main.yml** may consist of following tasks:
```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

### Step 4 – Reference ‘Webserver’ role
1. Within the **static-assignments** folder, create a new assignment for uat-webservers **uat-webservers.yml**. This is where you will reference the role.
```
---
- hosts: uat-webservers
  roles:
     - webserver
```
![step4pix1](https://user-images.githubusercontent.com/74002629/187206058-776ee522-a1f6-4934-9dc8-dfb987b3dad4.PNG)  

2. The entry point to the ansible configuration is the **site.yml** file. so, refer your uat-webservers.yml role inside site.yml in this format:
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```
![step4pix2](https://user-images.githubusercontent.com/74002629/187206078-e15187b6-9e8f-47a7-aedb-b4c22de12c6b.PNG)
  
### Step 5 – Commit & Test
1. Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into **/home/ubuntu/ansible-config-mgt/** directory.
![step4pix6](https://user-images.githubusercontent.com/74002629/187206146-9236d753-bcd1-4282-b593-0bfc183c8d52.PNG)  
  
2. Now run the playbook against your uat inventory and see what happens:
`sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml`

![pix8](https://user-images.githubusercontent.com/74002629/187206838-19510d55-419a-4ee6-9aca-811b498000cd.PNG)

3. You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser.
