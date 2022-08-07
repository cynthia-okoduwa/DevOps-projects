# LOAD BALANCER SOLUTION WITH APACHE
brief intro
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
6. Next, configure load balancing in the default config file: `sudo vi /etc/apache2/sites-available/000-default.conf`
7. In the config file add and save the following configuration into this section **<VirtualHost *:80>  </VirtualHost>** making sure to enter the IP of the webservers 
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
8. Restart Apache server: `sudo systemctl restart apache2`
9. Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:
`http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`
