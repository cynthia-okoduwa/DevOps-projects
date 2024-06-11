# Build a CICD Pipeline manually with Jenkins
### Project overview
The following are the objectives of this project:
1. Setup Jenkins along with Docker Server (DIND) as your Continuous Integration Environment, configure plugins, admin user and get Jenkins ready to start building your CI workflows.
2. Create a pipeline for a maven project.
3. Configure artifact versioning.
4. Connecting jobs with Upstreams and Downstreams
5. Create a pipeline view for your pipeline.
# Step 1
## Set up Jenkins server Docker(DIND)
There are many options to setting up a Jenkins enviroment such as setting it up on Windows or Linux based environments, on a Virtual machine etc. The set up I will be doing is setting Jenkins using Docker. This set up will allow you continously use your environment for as long as you want for your CICD practise.
All you need is to have Docker engine installed on your local machine or a VM and then you set up Jenkins server inside Docker.
1. To install Docker engine go to [Get Docker](https://docs.docker.com/get-docker/) and select the best installation path for you.
2. Start by validing your environment with:
```
docker version
docker-compose
```
The first command will show you both client and server information and the second command will return the help menu for docker compose, which validates that you have Doker installed, started and
docker-compose ready to go.

3.  Launch a jenkins container on your docker host by using recommended jenkins image similar to whats recommended in the official documentation [Docker](https://www.jenkins.io/doc/book/installing/docker/) using the following sequence ofcommands.
```
git clone https://github.com/udbc/bootcamp.git
cd bootcamp/jenkins
docker-compose up -d
```
The first command clones the repostory that would help you set up your enviromnet quickly. It is a declarative approach to setting up the enviromnet that you need. The next command changes your directory to where your jenkins folder is located. When you run the third command, it will read the Dockerfile, build the custom Jenkins image with all the required plugins(including Blue Ocean) and cofigurations.

4. Validate that Jenkins is setup along with Docker using the following command:
```
docker-compose ps
```
If all goes well you should see your running containers (Jenkins and Docker:dind)
![Screenshot 2024-06-07 152005](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/8454e410-a97a-45fd-88c4-f2a7d8fba3d2)

5. You can now access Jenkins UI by browsing to `http://localhost:8080`
![Screenshot 2024-06-07 152128](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/0125dfd1-86cb-4676-a2d0-12e10f2c0219)

6. To access Jenkins UI, you will need to provide the initial admin password. To get the admin password run the following command:
```
docker-compose logs jenkins
```
From the output copy the admin password as highlighted in the image below and paste in the Jenkins UI

![Screenshot 2024-06-07 152049](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/21d093c7-fa28-4d98-af5c-6a435848640f)

7. 
