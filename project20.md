## MIGRATION TO THE СLOUD WITH CONTAINERIZATION USING DOCKER

In this project, I demonstrate the process of migrating an application from a virtual machine to containers using Docker. A VM infrastructure requires an OS for
the host server and an additional OS for each hosted application. Because containers all share the underlying OS, a single OS can support more than one container. 
The elimination of extra operating systems means less memory, less drive space and faster processing so that applications run more efficiently.

#### STEP 1
To begin the project,  I created my Docker container on an Ubuntu 20.04 virtual machine. Below are the steps for installing Docker on Ubuntu 20.04 VM:
1. First, update your existing list of packages: `sudo apt update`
2. Install a few prerequisite packages which let apt use packages over HTTPS: `sudo apt install apt-transport-https ca-certificates curl software-properties-common`
3. Add the GPG key for the official Docker repository to your system: `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
4. Add the Docker repository to APT sources: `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"`
5. Make sure you are about to install from the Docker repo instead of the default Ubuntu repo: `apt-cache policy docker-ce`
6. You output should look something like this:
```
docker-ce:
  Installed: (none)
  Candidate: 5:19.03.9~3-0~ubuntu-focal
  Version table:
     5:19.03.9~3-0~ubuntu-focal 500
        500 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
```
7. Finally, install Docker: `sudo apt install docker-ce`
8. Check that that Docker is running: `sudo systemctl status docker`
9. Your output should show that Docker is active.
10. To get more information about installing Docker on Ubuntu 20.04, check this [link](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)
#### STEP 2 Create SQL Container and Connect to the container.
1. First, create a network. Creating a custom network is not mandatory, if we do not create a network, Docker will use the default network for all the containers. In this project however, the requirement is to control the cidr range of the containers running so I created a custom network with a specific cidr with the following code: ` sudo docker network create --subnet=172.18.0.0/24 tooling_app_network`
![pix4](https://user-images.githubusercontent.com/74002629/208447461-7107f1b9-96eb-4ddb-974d-cf9a64459b22.PNG)
2. Create an environment variable to store the root password: ` export MYSQL_PW=cynthiapw`
3. Echo enviroment variable to confirm it was created: `echo $MYSQL_PW` This would output the password you created.
4. Next, pull the image and run the container, all in one command like below:
` sudo docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest `
5. Verify the container is running: ` sudo docker ps -a`
![pix5](https://user-images.githubusercontent.com/74002629/208448222-d846880f-222c-4ee0-aa50-c44ed8f282f5.PNG)
6. Create an SQL script that to create a user that will connect remotely. Create a file and name it ****create_user.sql**** and add code in the file:
```
CREATE USER 'cynthia'@'%' IDENTIFIED BY 'cynthiapw';
GRANT ALL PRIVILEGES ON *.* TO 'cynthia'@'%';

CREATE DATABASE toolingdb;
```
The script also creates a database for the Tooling web application.
![pix6](https://user-images.githubusercontent.com/74002629/208448548-843a4e08-f288-4db2-a77b-3c237e13e5a4.PNG)

7. Run the script, ensure you are in the directory create_user.sql file is located or declare a path:
 `sudo docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql`
8. If you see a warning like below, it is acceptable to ignore: "mysql: [Warning] Using a password on the command line interface can be insecure"
9. Next, connect to the MySQL server from a second container running the MySQL client utility. To run the MySQL Client Container, type:
` sudo docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u  -p `
#### Step 3: Prepare database schema
1. Clone the Tooling-app repository from [here](https://github.com/darey-devops/tooling)
2. On your terminal, export the location of the SQL file: `export tooling_db_schema=/tooling_db_schema.sql`
3. Echo to verify that the path is exported: `echo $tooling_db_schema`
4.  Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.
` sudo docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema.sql`
![pix7](https://user-images.githubusercontent.com/74002629/208449253-8f74bc1a-ddbc-488d-beca-8d80cfa75d0f.PNG)

5. Update the `.env` file with connection details to the database. The .env file is located in the html **tooling/html/.env** folder but not visible in terminal. Use vi or nano
```
sudo vi .env

MYSQL_IP=mysqlserverhost
MYSQL_USER=cynthia
MYSQL_PASS=cynthiapw
MYSQL_DBNAME=toolingdb
```
![pix7](https://user-images.githubusercontent.com/74002629/208449253-8f74bc1a-ddbc-488d-beca-8d80cfa75d0f.PNG)

#### Step 4: Run the Tooling App
1. Before you run the tooling appication ensure you edit your security group to allow TCP traffic on port 8085 with access from anywhere(0.0.0.0)
2. In this project, I built my container from a pre-created Dockerfile located in the tooling directory. Navigate to the directory "tooling" that has the Dockerfile and build your container : ` sudo docker build -t tooling:0.0.1 . `
![pix8](https://user-images.githubusercontent.com/74002629/208449729-39489043-231b-406c-a260-89d2cf49966e.PNG)
3. Run the container: `docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1` 
4. Access your tooling site via: `http://<server-publicIP>:8085`
**Note:** I had an error that stated:AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.18.0.3. Set the 'ServerName' directive globally to suppress this message
I solved it by going into the db_conn.php file and hardcoded my enviroment variable. This is not the recommended way of doing it but this was how I got mine to work.
5. Get into the db_conn.php file: `sudo vi db_conn.php` and edit the Create connection variables.
![prob9](https://user-images.githubusercontent.com/74002629/208464715-5c919bb7-115e-419a-9fee-247aca874cbd.PNG)
6. Access your toolint site again: `http://<server-publicIP>:8085`
![pix13](https://user-images.githubusercontent.com/74002629/208465141-bba73216-8ff3-434b-9727-799f32b6c0ab.PNG)
![pix14](https://user-images.githubusercontent.com/74002629/208465156-510390a6-d568-44cf-93e5-ca1215aa1887.PNG)

