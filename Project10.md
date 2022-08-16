# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

we will configure an Nginx Load Balancer solution and also you will register your website with LetsEnrcypt Certificate Authority,
to automate certificate issuance you will use a shell client recommended by LetsEncrypt – cetrbot.

### Task 
This project consists of two parts:
1. Configure Nginx as a Load Balancer
2. Register a new domain name and configure secured connection using SSL/TLS certificates

#### Step 1 - CONFIGURE NGINX AS A LOAD BALANCER

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it **Nginx LB**, make sure to open TCP port 80 for HTTP connections and  
open TCP port 443 for secured HTTPS connections
2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses
3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers, update the instance and Install Nginx:
```
sudo apt update
sudo apt install nginx
```
4. Open the default nginx configuration file : `sudo vi /etc/nginx/nginx.conf`
5. insert following configuration into http section
```
 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }
```
6. Also in the configuration file, comment out this line:
`#       include /etc/nginx/sites-enabled/*;`
7. Restart Nginx and make sure the service is up and running
```
sudo systemctl restart nginx
sudo systemctl status nginx
```

#### Step 2 - REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

1. Register a domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)
2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP
3. Create a static IP address, allocate the Elastic IP and associate it with an EC2 server to ensure your IP remain the same everytime you restart the instance.
4. Update A record in your registrar to point to Nginx LB using Elastic IP address
Learn how associate your domain name to your Elastic IP on this page.

Side Self Study: Read about different DNS record types and learn what they are used for.

Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://<your-domain-name.com>

Configure Nginx to recognize your new domain name
Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com

Install certbot and request for an SSL/TLS certificate
Make sure snapd service is active and running

sudo systemctl status snapd
Install certbot

sudo snap install --classic certbot
Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).

sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>

You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.
Click on the padlock icon and you can see the details of the certificate issued for your website.
