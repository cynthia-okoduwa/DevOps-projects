
# TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. 
## INTRODUCTION TO JENKINS

#### Task
**Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.**

![Capture](https://user-images.githubusercontent.com/74002629/184101792-3c29fba2-78dd-4333-8aee-3385c605ecf1.PNG)

#### Step 1- Install Jenkins server
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
2. Install JDK (Java development kit) since Jenkins is a Java-based application:
```
sudo apt update
sudo apt install default-jdk-headless
```
3. Install Jenkins:
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
 ![pix2](https://user-images.githubusercontent.com/74002629/184062376-edb7e30b-8aca-475a-81bb-8db67da7e534.PNG)

4. Make sure Jenkins is up and running: `sudo systemctl status jenkins`
![pix3](https://user-images.githubusercontent.com/74002629/184062386-8be70305-bf29-46c2-b15e-0da44b9a1b1e.PNG)

5. By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
6. Next, setup Jenkins. From your browser access `http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080` You will be prompted to provide a default admin password
![pix4](https://user-images.githubusercontent.com/74002629/184062392-e1d57e72-b7eb-460f-8b7f-077555a88fae.PNG)

7. Retrieve the password from your Jenkins server: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
![pix5](https://user-images.githubusercontent.com/74002629/184062404-4aff3525-1fde-42fa-9bfa-07aad33ef129.PNG)

8. Copy the password from the server and paste on Jenkins setup to unlock Jenkins.
9. Next, you will be prompted to install plugins – **choose suggested plugins**
![pix6](https://user-images.githubusercontent.com/74002629/184062413-3c306670-0e55-4319-a5af-ff7757b0ff0e.PNG)

10. Once plugins installation is done – create an admin user and you will get your Jenkins server address. **The installation is completed!**
![pix7](https://user-images.githubusercontent.com/74002629/184062420-54d31942-052e-40f0-bcff-2d6820640868.PNG)


#### Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks
Here I configure a simple Jenkins job/project. This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings: 
```
Go to the tooling repository
Click on settings
Click on webhooks on the left panel
On the webhooks page under Payload URL enter: `http:// Jenkins server IP address/github-webhook`
Under content type select: application/json
Then add webhook
```
![pix8](https://user-images.githubusercontent.com/74002629/184062424-940204e5-ddb6-4d37-b667-659509933cbe.PNG)

2. Go to Jenkins web console, click **New Item** and create a **Freestyle project** and click OK
3. Connect your GitHub repository, copy the repository URL from the repository
4. In configuration of your Jenkins freestyle project under Source Code Management select **Git repository**, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.
![pix9](https://user-images.githubusercontent.com/74002629/184093372-97bdd653-2fc2-4940-9d19-4eff2386f370.PNG)

5. Save the configuration and let us try to run the build. For now we can only do it manually.
6. Click **Build Now** button, if you have configured everything correctly, the build will be successfull and you will see it under **#1**
7. Open the build and check in **Console Output** if it has run successfully.
![pix10](https://user-images.githubusercontent.com/74002629/184093378-af5503cd-77e0-4082-86df-72649418deaa.PNG)

This build does not produce anything and it runs only when it is triggered manually. Let us fix it.
8. Click **Configure** your job/project and add and save these two configurations:
``` 
Under **Build triggers** select: Github trigger for GITScm polling
Under **Post Build Actions** select Archieve the artifacts and enter `**` in the text box.
```
9. Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.
10. You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.
11. We have successfully configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub).
![pix16](https://user-images.githubusercontent.com/74002629/184093459-3d873ef8-6068-45d6-b56d-8d77669b5cf5.PNG)

12. By default, the artifacts are stored on Jenkins server locally: `ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

#### Step 3 – Configure Jenkins to copy files to NFS server via SSH
1. Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory. We need a plugin called
**Publish over SSh**
2. Install "Publish Over SSH" plugin.
3. Navigate to the dashboard select **Manage Jenkins** and choose **Manage Plugins** menu item.
4. On **Available** tab search for **Publish Over SSH** plugin and install it
![pix14](https://user-images.githubusercontent.com/74002629/184093437-ab971150-bb70-4393-a201-dc17617dd776.PNG)

5. Configure the job/project to copy artifacts over to NFS server.
6. On main dashboard select **Manage Jenkins** and choose **Configure System** menu item.
7. Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:
```
Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
Name- NFS
Hostname – can be private IP address of your NFS server
Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
```
![pix15](https://user-images.githubusercontent.com/74002629/184093450-be61c4f9-8214-4499-8db2-6dabafd1b954.PNG)

8. Test the configuration and make sure the connection returns **Success** Remember, that TCP port 22 on NFS server must be open to receive SSH connections.
9. Save the configuration and open your Jenkins job/project configuration page and add another one Post-build Action: **Set build actionds over SSH**
10. Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories – so we use **
![pix17](https://user-images.githubusercontent.com/74002629/184093471-5c12b087-5427-4c2a-b205-b7fb17f78de6.PNG)

11. Save this configuration and go ahead, change something in **README.MD** file the GitHub Tooling repository.
12. Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:
```
SSH: Transferred 25 file(s)
Finished: SUCCESS
```
![pix19](https://user-images.githubusercontent.com/74002629/184093501-6b1ed16a-66d0-4cb1-be0a-e5ba12a69442.PNG)
13. To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to your NFS server and check README.MD file: `cat /mnt/apps/README.md`
14. If you see the changes you had previously made in your GitHub – the job works as expected.
![pix18](https://user-images.githubusercontent.com/74002629/184095205-e37fa908-b2bf-4286-b553-afa26113175d.PNG)


#### Issues
1. After step 11, I got a "Permission denied" error which indicated that by build was not successful
2. I fixed the issue by changing mode and ownership on the NFS server with the following:
```
ll /mnt
sudo chown -R nobody:nobody /mnt
sudo chmod -R 777 /mnt
```




