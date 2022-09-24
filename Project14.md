## END-TO-END IMPLEMENTATION OF CI/CD PIPELINE FOR A PHP BASED APPLICATION

##### Prerequsites
1. Servers: You will require 6 servers for the project which incudes:
- nginx server: This would act as the reverse proxy server to our site and tool. 
- Jenkins server: To be used to implement your CI/CD workflows or pipelines. Select a t2.medium at least, Ubuntu 20.04 and Security group should be open to port 8080
- SonarQube server: To be used for Code quality analysis. Select a t2.medium at least, Ubuntu 20.04 and Security group should be open to port 9000
- -Artifactory server: To be used as the binary repository where the outcome of your build process is stored. Select a t2.medium at least and Security group should be open to port 8081
- Database server:
- Todo webserver:
(For the purposes of this project, you can have create one security group that is open to all traffic bearing in mind that this is not )
2. Your Ansible inventory should look like this  
```
├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat
```
focus will be mainly on the CI, Dev and Pentest Enviroments 

3. Ansible roles for the CI environment. In addition to the previous Ansible roles from project 13, in your ansibile-config-mgt repo add 2 more roles: [Sonarqube](https://www.sonarqube.org/) and [Artifactory](https://jfrog.com/artifactory/).
4. Configure Ansible For Jenkins Deployment. See [Project 9](https://github.com/cynthia-okoduwa/DevOps-projects/blob/main/Project9.md) for the initial setup of Jenkins. Here I will be comfiguring Jenkins to run Ansible commands in Jenkins UI.
- Navigate to Jenkins URL: `<Jenkins-server-public-IP>:8080
- In the Jenkins dashboard, click on manage plugins and search for Blue Ocean plugin. Install and open Blue Ocean plugin.
- In the Blue Ocean UI create a new pipeline.
- Select GitHub as where you store your code.
- Next enter your access token or create one if you don't have one, then enter the newly create Access token. 
