# LOAD BALANCER SOLUTION WITH APACHE
![Capture](https://user-images.githubusercontent.com/74002629/183334671-0641051c-31e2-44e9-950c-b2f7197b6343.PNG)
### Step 1 Configure Apache As A Load Balancer 
1. Create an Ubuntu Server 20.04 EC2 instance and name it **Project-8-apache-lb**.
2. Open TCP port 80 on **Project-8-apache-lb** by creating an Inbound Rule in Security Group.
3. Connect to the server through the SSh terminal and install Apache Load balancer then configure it to point traffic coming to LB to the Web Servers by running the following:
```
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev
```
4. Enable the following and restart the service:
```
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

sudo systemctl restart apache2
```
5. Ensure Apache2 is up and running: `sudo systemctl status apache2`
![pix1](https://user-images.githubusercontent.com/74002629/183334681-752ce1e8-cf63-4a09-9995-9693c01b1b3d.PNG)

7. Next, configure load balancing in the default config file: `sudo vi /etc/apache2/sites-available/000-default.conf`
8. In the config file add and save the following configuration into this section **<VirtualHost *:80>  </VirtualHost>** making sure to enter the IP of the webservers 
```
<Proxy "balancer://mycluster">
               BalancerMember http://172.31.95.40:80 loadfactor=5 timeout=1
               BalancerMember http://172.31.89.249:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```
![pix2](https://user-images.githubusercontent.com/74002629/183334693-52064187-9d6f-4c4e-a546-019e97de0fb3.PNG)

8. Restart Apache server: `sudo systemctl restart apache2`
9. Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:
`http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`
![pix3](https://user-images.githubusercontent.com/74002629/183334724-8419040e-1711-4783-96a9-277ec2c58145.PNG)
* The load balancer accepts the traffic and distributes it between the servers according to the method that was specified.
10. In Project-7 I had mounted **/var/log/httpd/** from the Web Servers to the NFS server, here I shall unmount them and give each Web Server has its own log directory: `sudo umount -f /var/log/httpd`
11. Open two ssh/Putty consoles for both Web Servers and run following command: `sudo tail -f /var/log/httpd/access_log`
12.  Refresh the browser page `http://172.31.95.118/index.php` with the load balancer public IP several times and make sure that both servers receive HTTP GET requests from your LB – new records will appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be disctributed evenly between them.
![pix4](https://user-images.githubusercontent.com/74002629/183334734-dbae496e-d27d-49f4-b01e-850eb49fd9ba.PNG)
![pix5](https://user-images.githubusercontent.com/74002629/183334749-4f5c17f2-c9e9-4034-9e6e-72e7dbab65ff.PNG)

### Step 2 – Configure Local DNS Names Resolution
1. Sometimes it may become tedious to remember and switch between IP addresses, especially when you have a lot of servers under your management.
We can solve this by configuring local domain name resolution. The easiest way is to use **/etc/hosts file**, although this approach is not very scalable, but it is very easy to configure and shows the concept well. 
2. Open this file on your LB server: `sudo vi /etc/hosts`
3. Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers
```
172.31.95.40 Web1
172.31.89.249 Web2
```
![pix6](https://user-images.githubusercontent.com/74002629/183334761-4df087b0-b6b2-4bce-a63a-8ab34a9578f6.PNG)

3. Now you can update your LB config file with those names instead of IP addresses.
```
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```
![pix7](https://user-images.githubusercontent.com/74002629/183334771-6e274762-6a6f-4d35-b6d9-48929322cbd3.PNG)

4. You can try to curl your Web Servers from LB locally `curl http://Web1` or `curl http://Web2` to see the HTML formated version of your website.
![pix8](https://user-images.githubusercontent.com/74002629/183334784-ef5e63ba-78d2-4241-892b-b6669940d54c.PNG)
![pix9](https://user-images.githubusercontent.com/74002629/183334797-ad9753e0-d34b-47f8-9146-66ff4c9de5f0.PNG)

