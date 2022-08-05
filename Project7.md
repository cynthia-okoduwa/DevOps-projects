# DEVOPS TOOLING WEBSITE SOLUTION
![Capture](https://user-images.githubusercontent.com/74002629/183053774-9dddd124-bdb1-4e78-b077-e5877b85fb33.PNG)

### Prerequites
1. Provision 4 Red Hat Enterprise Linux 8. One will be the NFS server and the other as the Web servers.
2. Provision 1 Ubuntu 20.04 for the the databaes server.

### Step 1 - Prepare NFS server
1. To view all logical volumes, run the command `lsblk` The 3 newly created block devices are names **xvdf**, **xvdh**, **xvdg** respectively.
2. Use gdisk utility to create a single partition on each of the 3 disks `sudo gdisk /dev/xvdf`
3. A prompt pops up, type `n` to create new partition, enter no of partition(1), hex code is 8300, `p` to view partition and `w` to save newly created partition.
4. Repeat this process for the other remaining block devices.
5. Type lsblk to view newly created partition.
6. Install lvm2 package by typing: `sudo yum install lvm2` then run `sudo lvmdiskscan` command to check for available partitions.
![Pix1](https://user-images.githubusercontent.com/74002629/183050422-48ff7ae2-982d-4ac7-9cf2-254a123a860c.PNG)

8. Create physical volume to be used by lvm by using the pvcreate command:
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
![pix2](https://user-images.githubusercontent.com/74002629/183050437-d9a55dbb-ca1d-4b6f-8bb5-5f2c5bd1aa72.PNG)

8. To check if the PV have been created successfully, run: `sudo pvs`
9. Next, Create the volume group and name it webdata-vg: `sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`
10. View newly created volume group type: `sudo vgs`
11. Create 3 logical volumes using lvcreate utility. Name them: lv-apps for storing data for the website, lv-logs for storing data for logs and lv-opt for Jenkins Jenkins server in project 8.
```
sudo lvcreate -n lv-apps -L 9G webdata-vg
sudo lvcreate -n lv-logs -L 9G webdata-vg
sudo lvcreate -n lv-opt -L 9G webdata-vg
```
![pix5](https://user-images.githubusercontent.com/74002629/183050487-41f518eb-ffcb-46a2-84e0-36839d51b6ed.PNG)

12. Verify Logical Volume has been created successfully by running: `sudo lvs`
13. Next, format the logical volumes with ext4 filesystem:
```
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
```
![pix7](https://user-images.githubusercontent.com/74002629/183050528-14284dad-e0f5-4858-8ed5-b9fd29c12032.PNG)

14. Next, create mount points for the logical volumes. Create **/mnt/apps** the following directory to store website files: 
```
sudo mkdir /mnt/apps
sudo mkdir /mnt/logs
sudo mkdir /mnt/opt
```
15. Mount to **/dev/webdata-vg/lv-apps** **/dev/webdata-vg/lv-apps** and **/dev/webdata-vg/lv-opt** respectievly : 
```
sudo mount /dev/webdata-vg/lv-apps /mnt/apps
sudo mount /dev/webdata-vg/lv-logs /mnt/logs
sudo mount /dev/webdata-vg/lv-opt /mnt/opt
```
16. Install NFS server, configure it to start on reboot and make sure it is up and running
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
![pix8](https://user-images.githubusercontent.com/74002629/183051438-77d0ecbe-0812-487a-b754-b2837a67e7e3.PNG)
17. Export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security.
18. Set up permission that will allow our Web servers to read, write and execute files on NFS:
```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```
![pix9](https://user-images.githubusercontent.com/74002629/183051442-69ca2423-75d4-4b3c-9ac1-afe5ee373b0b.PNG)
19. In your choosen text editor, configure access to NFS for clients within the same subnet (my Subnet CIDR – 172.31.80.0/20 ):
```
sudo vi /etc/exports

/mnt/apps 172.31.80.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.80.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.80.0/20(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv
```
20. Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)
`rpcinfo -p | grep nfs`
![pixSG](https://user-images.githubusercontent.com/74002629/183053344-f40f0d65-5670-4613-835c-1da0137e0416.PNG)

### STEP 2 — CONFIGURE THE DATABASE SERVER
1. Install and configure a MySQL DBMS to work with remote Web Server
2. SSH in to the provisioned DB server and run an update on the server: `sudo apt update`
3. Install mysql-server: `sudo apt install mysql-server -y`
4. Create a database and name it **tooling**: 
```
sudo my sql
create database tooling;
```
5. Create a database user and name it **webaccess** and grant permission to **webaccess** user on tooling database to do anything only 
from the webservers subnet cidr:
```
create user 'webaccess'@'172.31.80.0/20' identified by 'password';
grant all privilleges on tooling.* to 'webaccess'@'172.31.80.0/20';
flush privileges;
```
6. To show database run: `show databases;`
![pix10](https://user-images.githubusercontent.com/74002629/183051459-c2a2c22e-44ec-453b-9d2b-44000ceccae1.PNG)

### Step 3 — Prepare the Web Servers

1. Install NFS client on the webserver1: `sudo yum install nfs-utils nfs4-acl-tools -y`
2. Mount /var/www/ and target the NFS server’s export for apps (Use the private IP of the NFS server)
```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid 172.31.85.14:/mnt/apps /var/www
```
3. Verify that NFS was mounted successfully by running `df -h` Make sure that the changes will persist on Web Server after reboot:
`sudo vi /etc/fstab`
4. Add the following line in the configuration file: `172.31.85.14:/mnt/apps /var/www nfs defaults 0 0`
5. Install Remi’s repository, Apache and PHP:
```
sudo yum install httpd -y
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo dnf module reset php
sudo dnf module enable php:remi-7.4
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
6. Repeat steps 1-5 for the other 2 webservers
7. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps.
![pix11](https://user-images.githubusercontent.com/74002629/183030599-7de0f18a-8050-4e42-bf6d-3307d8ff0ac7.PNG)

8. Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №3 and №4 to make sure the mount point will persist after reboot:
```
sudo mount -t nfs -o rw,nosuid 172.31.85.14:/mnt/logs /var/log/httpd
sudo vi /etc/fstab
172.31.85.14:/mnt/logs /var/log/httpd nfs defaults 0 0
```
9. Fork the tooling source code from **Darey.io** Github Account to your Github account. 
10. Begin by installing git on the webserver: `sudo yum install git -y`
11. Initialize Git: `git init`
12. Then run: `git clone https://github.com/darey-io/tooling.git`
![pix12](https://user-images.githubusercontent.com/74002629/183052304-f20cf002-f862-42c2-b71c-a8619b16caaf.PNG)

13. Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html
![pix13](https://user-images.githubusercontent.com/74002629/183036121-425bca5c-d3dc-442c-8943-d9134077b4b2.PNG)

14. On the webserver, ensure port 80 in open to all traffic in the security groups.
15. Update the website’s configuration to connect to the database: `sudo vi /var/www/html/functions.php`
![pix15](https://user-images.githubusercontent.com/74002629/183052617-0bc05ae2-997f-4384-a157-558d707f34f0.PNG)

16. Apply tooling-db.sql script to your database using this command `mysqli_connect ('172.31.80.140', 'webaccess', 'password', 'tooling')`
17. In the databse server update the bind address to 0.0.0.0: `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
18. Then create in MySQL a new admin user with username: myuser and password: password:
```
INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);
```
![pix16](https://user-images.githubusercontent.com/74002629/183053301-e021a287-f188-441e-96d0-022663e79a2d.PNG)
Finally, open the website in your browser with the public IP of the webserver and make sure you can login into the websute with myuser user.
![pix17](https://user-images.githubusercontent.com/74002629/183053304-87db560b-8cbb-448e-96e1-caae58c2f0c1.PNG)
![pix18](https://user-images.githubusercontent.com/74002629/183053318-df3c9915-a54d-46d2-b798-5f54e3d8bb43.PNG)
