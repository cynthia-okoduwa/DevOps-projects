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

### Phase 1
##### Prepare your Jenkins server
1. Set up SSH-agent:
2. Connect to the instance on VScode.
3. Install the following packages and dependencies on the server:
- Install git :
- Clone dwn the Asible-config-mgt repository: `git clone 
- Install Jenkins and its dependencies. Steps to install Jenkins can be found [here](https://www.jenkins.io/doc/book/installing/)
4. Configure Ansible For Jenkins Deployment. See [Project 9](https://github.com/cynthia-okoduwa/DevOps-projects/blob/main/Project9.md) for the initial setup of Jenkins. Here I will be comfiguring Jenkins to run Ansible commands in Jenkins UI.
- Navigate to Jenkins URL: `<Jenkins-server-public-IP>:8080`
- In the Jenkins dashboard, click on Manage Jenkins -> Manage plugins and search for Blue Ocean plugin. Install and open Blue Ocean plugin.
![pix1](https://user-images.githubusercontent.com/74002629/192139875-9d78fb62-afd5-4999-b8a8-0c40e5acca34.PNG)

- In the Blue Ocean UI create a new pipeline.
![pix2](https://user-images.githubusercontent.com/74002629/192139879-ad7f7142-ac78-473e-bcc0-2e374e16d4e1.PNG)

- Select GitHub as where you store your code.
![pix3](https://user-images.githubusercontent.com/74002629/192139882-c6beac02-30eb-4a06-8206-41f087948fc4.PNG)

- Create Access token, then enter the newly create Access token. Login to GitHub & Generate an Access
![pix4](https://user-images.githubusercontent.com/74002629/192139886-5f9d8281-2222-454a-9563-73711414fecc.PNG)
 
- Copy Access token and paste in the new pipeline, then connect.
- Select which organisation the repository belongs to.
![pix5](https://user-images.githubusercontent.com/74002629/192139891-99ef48d7-e64e-4f41-988a-155e3724147b.PNG)

- At this point you do not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. We do not need that, rather we will create one ourselves. So, click on Administration to exit the Blue Ocean console.
- In our Jenkins dashboard you will find the newly created pipeline.
![pix6](https://user-images.githubusercontent.com/74002629/192139894-50db210d-d148-4cb5-8a0d-f830909ab592.PNG)

5. Let us create our Jenkinsfile.
- In Vscode, inside the Ansible project, create a new directory and name it **deploy**, create a new file Jenkinsfile inside the directory.
![pix7](https://user-images.githubusercontent.com/74002629/192139899-94eb40be-fd85-467e-b72e-e5a2d699ed3b.PNG)

- Add the code snippet below to start building the test Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage
```
pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```
6. Next go back into the Ansible pipeline in Jenkins, and select configure
7. Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile
![pix9](https://user-images.githubusercontent.com/74002629/192139913-1e03fd48-6c25-4686-94a2-243940d8795a.PNG)

8.Back to the pipeline again, this time click "Build now"
![pix10](https://user-images.githubusercontent.com/74002629/192139916-1f43005f-cba7-42e5-823d-0bfa70c688bb.PNG)

9. This will trigger a build and you will be able to see the effect of our test Jenkinsfile configuration by going through the console output of the build.
To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface. Click on Blue Ocean
10. Select your project and click on the play button against the branch
11. This pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch. To see this in action: Create a new git branch and name it **feature/jenkinspipeline-stages**
Currently we only have the Build stage. Let us add another stage called Test. Paste the code snippet below and push the new changes to GitHub.
```
 pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```
12. To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository. Click on the "Administration" button and navigate to the Ansible project and click on "Scan repository now"
13. Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.
14. In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.

### Phase 2
1. Install Ansible on Jenkins your ubuntu VM. Follow the steps in this link to insall [Ansible](https://www.cyberciti.biz/faq/how-to-install-and-configure-latest-version-of-ansible-on-ubuntu-linux/)
2. Install Ansible plugin in Jenkins UI
Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully) Note: Ensure that Ansible runs against the **Dev** environment successfully.

**Some possible errors to watch out for:**
- Ensure that the git module in Jenkinsfile is checking out SCM to **main** branch instead of **master** (GitHub has discontinued the use of Master due to Black Lives Matter)
- Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the .ansible.cfg file alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set.

