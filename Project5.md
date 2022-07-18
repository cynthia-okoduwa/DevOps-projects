## Client/Server Architecture Using A MySQL Relational Database Management System
### TASK – Implement a Client Server Architecture using MySQL Database Management System (DBMS).
#### Steps
1. Spin up two Linux-based virtual servers (EC2 instances in AWS) and name them: `mysql server` and `mysql client` respectively.
2. Run `sudo apt update -y` for lastest updates on server.
![Pix1](https://user-images.githubusercontent.com/74002629/179509562-143bc321-9064-4788-96c5-23d99e76931c.PNG)

3. Next, on **mysql server** install MySQL Server software: `sudo apt install mysql-server -y`
4. On **mysql client** install MySQL Client software: `sudo apt install mysql-client -y`
![pix2](https://user-images.githubusercontent.com/74002629/179509582-c75aee5c-e666-420a-9c95-d3f4b9318d5e.PNG)

5. Edit Inbound rule on **mysql server** to allow access to **mysql client** traffic. MySQL server uses TCP port 3306 by default. Specify inbound traffic from the IP
of **mysql cient** for extra security.
![pix 3](https://user-images.githubusercontent.com/74002629/179509596-05ed6043-44db-4801-82bc-55bcbd06711f.PNG)

6. For **mysql client** to gain remote access to **mysql server** we need to create and database and a user on **mysql server**. To start with run the mysql security 
script: `sudo mysql_secure_installation` Follow the prompts and answer appropraitely to finish the process.
![pix4](https://user-images.githubusercontent.com/74002629/179509619-5350cff4-ae12-4c5a-adb7-67909c0cf209.PNG)

7. Run mysql command: `sudo mysql` This would take you to the mysql prompt (You may be required to input password if you opten for the validate password during 
the security script installation)
8. Next, create the remote user with this following command: `CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';`
![pix5](https://user-images.githubusercontent.com/74002629/179509648-192f958c-588d-485c-8fbd-b82b087f28f0.PNG)

9. Create database with: `CREATE DATABASE test_db;`
10.  Then grant privieges to remote_user:  `GRANT ALL ON test_db.* TO 'remote_user'@'%' WITH GRANT OPTION;`
11.  . Finally, flush privileges and exit mysql : `FLUSH PRIVILEGES;`
![PIX7](https://user-images.githubusercontent.com/74002629/179509685-0d7cc63a-b82d-4335-a71a-31d723095d73.PNG)

12. Having created the user and database, configure MySQL server to allow connections from remote hosts. Use the following command: `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
13. In the text editor, replace the old **Bind-address** from ‘127.0.0.1’ to ‘0.0.0.0’ then save and exit.
![pix10](https://user-images.githubusercontent.com/74002629/179512615-0c5a49e2-c66d-4a2e-9214-452c726e25bb.PNG)

14. Next, we restart mysql with: `sudo systemctl restart mysql`
15. From **mysql client** connect remotely to **mysql server** Database Engine without using SSH. Using the mysql utility to perform this action type:
`sudo mysql -u remote_user -h 172.31.3.70 -p` and enter `password` for the user password.
![pix11](https://user-images.githubusercontent.com/74002629/179512631-8e14a82c-7cd2-41f7-98d1-3a20791605f7.PNG)

16. This gives us access into the mysql server database engine.
17. Finally type: `Show databases;` to show the test_db database that was created.
![pix12](https://user-images.githubusercontent.com/74002629/179512639-524ee577-45b3-4db1-9c17-37d8b2679268.PNG)
