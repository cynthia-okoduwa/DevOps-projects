# Build a CICD pipeline manually with Jenkins
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
From the output copy the admin password as highlighted in the image below and paste in the Jenkins UI for access

![Screenshot 2024-06-07 152049](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/21d093c7-fa28-4d98-af5c-6a435848640f)

7. Next, choose "install suggested plugins" to configure the default plugins automatically. Once plugins have been successfully installed, create your Jenkins Admin user. This would provide you with your Jenkins configuration page.

![Screenshot 2024-06-07 152648](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/83e3070e-15ac-418f-bf56-835b219a88a8)

![Screenshot 2024-06-07 152825](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/7a148e74-a288-4cfd-b3bd-bc6b61241e8c)

## Starting, stopping and resetting Jenkins
When setting up your Continuous Integration workflows, there will be instances where you need to pause and resume your environment, such as when shutting down your laptop at the end of the day and restarting it the next day. To ensure your learning lab environment is up and running again, you can gracefully stop the lab environment by using:
```
docker-compose stop
```
You can restart your enviroment by using:
```
docker-compose up -d
```

# Step 2
## Set up pipeline for Maven project.

In this step, I will begin the process of building a pipeline for a java application that uses Maven as a build tool. Before starting to build the pipeline, first fork the Github repository of the application code [here](https://github.com/udbc/sysfoo)

### Set up Build Job

1. Go to *Manage Jenkins* then click on *Global tools* configuration and under *Maven* section, provide name as **Maven** and select the maven version
**3.9.6** the save the changes.

![Screenshot 2024-06-09 061346](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/e62a9f19-dff2-4ff0-962a-0521748c5794)

2. Next. install **Maven integration plugin**. Go to **Manage Jenkins** then click on **Manage Plugins**, **Available plugins**, search for **Maven Integration plugin** and install it without restart. Once installed click on **Go back to the top page** to return to Jenkins main page.
3. From the Jenkins main page, create a new folder that will serve as a namespace. Do so by creating a **new item**, with type **folder** with name **sysfoo** then click **ok**.
4. On the next page, leave all other configurations as is and click save to proceed. That creates a folder by name sysfoo.

![Screenshot 2024-06-09 062049](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/a07d037d-5221-44f5-845e-66e464628f97)

5. Inside the sysfoo folder, Create a new job and select **Maven project** as the type of project. Name the job as **build**
6. Next, go to source code management, enter the URL of your git repository that you forked from GitHub for this project, then change the **branch specifier** to `*/main`

![Screenshot 2024-06-09 062812](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/f7a8701b-5007-4845-a209-dc36031932af)

7. Click on **build** on the left hand panel, it would take you to the build configurations. Provide your path to *pom.xml*. In this example the file is inside the root directory of sysfoo, so nothing to change. Go to build step and provide the goal and option as *compile*.

![Screenshot 2024-06-14 124803](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/ebffb65e-fd99-49fa-a923-bcec0707edb5)

8. Save the job and click on **Build Now**. Observe the job status, console output etc.

### Adding Unit test and Packaging jobs
After creating the Build job, its time to add Test and Package jobs for our sysfoo application.
1. Start by creating a new job inside sysfoo folder and name it Test.
2. Scroll down to the bottom of the page, in the *Copy from* field, enter *build* this will ensure that everything configured in the build job is automatically reflected in the test job.

![Screenshot 2024-06-11 075245](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/8c83e785-4938-43e7-8020-de8a42901497)

3. In test job , change your description, in this example i set it as *test sysfoo java app* . Leave the *Source code management* repository as is.
4. Under build step, change the goals and option to *clean test* and leave the other fields as is. Save the job and build.

Next job is package , this will compile the application and then generate the war file. 
1. Create a job on same folder with the name of package , and copy configurations from the test job.

![Screenshot 2024-06-11 075657](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/621204f3-206d-4c67-a3f8-cd8e7735feaf)

2. Update the description as *package sysfoo java app*, create jar and only change in the configuration is build step.
3. In the build step for the package job, change the goal to *package -DskipTests* , save the changes and build.

![Screenshot 2024-06-11 075842](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/65ff7b32-19ea-4b4b-8404-71e33b904db4)

4. After a build successful, It will create a *jar* file, you will find the jar file created in the *Workspace* target directory. Verify the jar file is created by clicking on workspace

![Screenshot 2024-06-11 080731](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/87c92b6b-d2f3-4969-804a-7a13d32bd61f)

5. You can publish or acrhive the jar file for future use. Configure the package from the **post build actions**, choose **archive the artifacts** and provide path `**target/*.jar` in the files to archive input field so that the jar file created at this path (e.g. sysfoo.war) is automatically archived/published.

![Screenshot 2024-06-15 103843](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/315fd76e-78be-4647-bccc-50fc7879ab30)

6. Save the changes and build the job. Once build is successful, check the project page to find out your artifacts right out there.

![Screenshot 2024-06-11 082600](https://github.com/cynthia-okoduwa/DevOps-projects/assets/74002629/7efc0f7f-42e1-4728-a470-84ad909cafb5)

7.



