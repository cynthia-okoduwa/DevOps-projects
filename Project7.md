# DEVOPS TOOLING WEBSITE SOLUTION

### Step 1 -
1. To view all logical volumes, run the command `lsblk` The 3 newly created block devices are names **xvdf**, **xvdh**, **xvdg** respectively.
2. Use gdisk utility to create a single partition on each of the 3 disks `sudo gdisk /dev/xvdf`
3. A prompt pops up, type `n` to create new partition, enter no of partition(1), hex code is 8300, `p` to view partition and `w` to save newly created partition.
4. Repeat this process for the other remaining block devices.
5. Type lsblk to view newly created partition.
6. Install lvm2 package by typing: `sudo yum install lvm2` then run `sudo lvmdiskscan` command to check for available partitions.
7. Create physical volume to be used by lvm by using the pvcreate command:
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
8. To check if the PV have been created successfully, run: `sudo pvs`
9. Next, Create the volume group and name it webdata-vg: `sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`
10. View newly created volume group type: `sudo vgs`
11. Create 3 logical volumes using lvcreate utility. Name them: lv-apps for storing data for the website, lv-logs for storing data for logs and lv-opt for Jenkins Jenkins server in project 8.
```
sudo lvcreate -n lv-apps -L 9G webdata-vg
sudo lvcreate -n lv-logs -L 9G webdata-vg
sudo lvcreate -n lv-opt -L 9G webdata-vg
```
12. Verify Logical Volume has been created successfully by running: `sudo lvs`
13. Next, format the logical volumes with ext4 filesystem:
```
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
```
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

### STEP 2 — CONFIGURE THE DATABASE SERVER
1. Install and configure a MySQL DBMS to work with remote Web Server
2. SSH in to the provisioned DB server and run an update on the server: `sudo apt update`
4. Install mysql-server: `sudo apt install mysql-server -y`
5. Create a database and name it **tooling**: 
```
sudo my sql
create database tooling;
```
6. Create a database user and name it **webaccess** and grant permission to **webaccess** user on tooling database to do anything only 
from the webservers subnet cidr:
```
create user 'webaccess'@'172.31.80.0/20' identified by 'password';
grant all privilleges on tooling.* to 'webaccess'@'172.31.80.0/20';
flush privileges;
```
7. To show database run: `show databases;`
7. 
