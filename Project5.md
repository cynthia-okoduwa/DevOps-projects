## Client/Server Architecture Using A MySQL Relational Database Management System
### TASK – Implement a Client Server Architecture using MySQL Database Management System (DBMS).
#### Steps





1. Spin up two Ubuntu virtual servers (EC2 Instances In AWS) and name them: mysql server and  mysql client respectively

![Screenshot from 2022-10-04 16-24-35](https://user-images.githubusercontent.com/46121207/193996693-af437184-76cf-4bec-aa18-6d45ecea70ce.png)


2. ssh into the mysql server 
  sudo apt update -y
  ![Screenshot from 2022-10-04 17-45-57](https://user-images.githubusercontent.com/46121207/193997004-92be8f10-53a5-444c-b033-97275de7ddfc.png)



3. Next, on mysql server install MYSQL Server software: sudo apt install mysql-server -y

![Screenshot from 2022-10-04 17-46-45](https://user-images.githubusercontent.com/46121207/193997157-689cd0f8-de94-4098-a82e-98b6e4415059.png)


4. Edit Inbound rule on mysql server to allow access to mysql client traffic. MySQL server uses TCP port 3306 by default. Specify inbound traffic from the IP of mysql cient for extra security.

Note: Basically, it means one should copy the  private ip address of the mysql-client and paste it as an inbound rule in mysql server security group.  


![Screenshot from 2022-10-04 16-41-21](https://user-images.githubusercontent.com/46121207/193996765-1153fd88-8ed2-4025-b27d-a31ba375de70.png)

Extra Tips: If you decide to use 0.0.0.0/0 in mysql/Aurora, it means in step 12, you will copy the public Ip address of your mysql-server as the ip address when you ssh in your mysql-client server.  You will understand better when you finish the lab.


![Screenshot from 2022-10-05 07-34-45](https://user-images.githubusercontent.com/46121207/193996777-7da3b66e-43e3-4ccf-b427-ffc58c1e1ef8.png)



5. Run mysql command:  sudo mysql -u root -p This would take you to the mysql prompt( you may be prompted to input password)
Then run: ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'mynewpassword'; to alter the root user  
![Screenshot from 2022-10-05 07-52-48](https://user-images.githubusercontent.com/46121207/193998688-eb6a9af9-2a5f-402e-8e62-28305bddf8a7.png)


![Screenshot from 2022-10-05 06-35-45](https://user-images.githubusercontent.com/46121207/193997344-e20212cf-07d7-4f0d-832e-3adb22e5b713.png)

Then type:  exit




6. For mysql client to gain remote access to mysql server we need to create and database and a user on mysql server. To start with run the mysql security script: sudo mysql_secure_installation Follow the prompts and answer appropraitely to finish the process.

	![Screenshot from 2022-10-05 06-40-28](https://user-images.githubusercontent.com/46121207/193997419-08c3190b-14c7-4280-af4b-eb7289af24e3.png)



7. Next, create the remote user with this following command: CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';

![Screenshot from 2022-10-05 06-43-31](https://user-images.githubusercontent.com/46121207/193997469-b48e9c81-76ac-4ab8-9acc-32eaeef5420e.png)

8. Create a database using: CREATE DATABASE test_db; 
To grant privileges:  GRANT ALL ON test_db.* TO 'remote_user'@'%' WITH GRANT OPTION; 

Flush privileges and exit mysql:  FLUSH PRIVILEGES; 
![Screenshot from 2022-10-05 05-30-04](https://user-images.githubusercontent.com/46121207/193997574-c5aa8e56-2a33-4ee3-a55f-80a957206833.png)

 9. Having created the user and database, configure MySQL server to allow connections from remote hosts. Use the following command: sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf 
 


 In the text editor, replace the old **Bind-address** from ‘127.0.0.1’ to ‘0.0.0.0’ then save and exit.


![Screenshot from 2022-10-05 05-33-38](https://user-images.githubusercontent.com/46121207/193997946-02fc658b-c17e-4a64-a102-3a0c1b5c0411.png)


10. Next, we restart mysql with: 
sudo systemctl restart mysql

11. SSH into your mysql client, Run:
 sudo apt update -y
 
RUN this code to install MYSQLClient:
sudo apt install mysql-client -y



12. From mysql client connect remotely to mysql server Database Engine without using SSH. Using the mysql utility to perform this action type:

sudo mysql -u remote_user -h **mysql-server private ip** -p and enter `password` for the user password

Note: use the password in the step 7, which is ‘password’

  
![Screenshot from 2022-10-05 07-31-10](https://user-images.githubusercontent.com/46121207/194000799-4593ed7a-9aca-470f-810d-96e853242dfa.png)

  


