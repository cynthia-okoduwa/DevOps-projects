# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

In this project, we will solidify our knowledge of load balancers and make us versatile in our knowledge of configuring a different type of LB. We will configure an Nginx Load Balancer solution and also register our website with LetsEnrcypt Certificate Authority, to automate certificate issuance. 

A certificate is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. In our project we will use a shell client recommended by LetsEncrypt – cetrbot.

Our achitecture will look something like this:

![pix18](https://user-images.githubusercontent.com/74002629/184848922-0b777f13-bef5-4361-9a97-a3996c451f3e.PNG)

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
![pix4](https://user-images.githubusercontent.com/74002629/184851120-9fc08d7a-f638-4b85-b57d-6fdaa543fc50.PNG)

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
4. Update **A record** in your registrar to point to Nginx LB using Elastic IP address

![pix8](https://user-images.githubusercontent.com/74002629/184852049-7edb350f-1061-4e95-b545-4ec159324986.PNG)

6. Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://buildwithme.link

![pix7](https://user-images.githubusercontent.com/74002629/184852273-381a3fdc-b150-4452-b01e-c062067456da.PNG)

8. Configure Nginx to recognize your new domain name, update your nginx.conf with server_name www.buildwithme.link instead of server_name www.domain.com
9. Next, install certbot and request for an SSL/TLS certificate, first install certbot dependency: `sudo apt install python3-certbot-nginx -y`
10. Install certbot: `sudo apt install certbot -y`
12. Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).
```
sudo certbot --nginx -d biuldwithme.link -d www.buildwithme.link
```
![pix12](https://user-images.githubusercontent.com/74002629/184856111-572b3705-6232-4b20-ae1a-c85534e8fb1a.PNG)

10. Test secured access to your Web Solution by trying to reach https://buildwithme.link, if successful, you will be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string. Click on the padlock icon and you can see the details of the certificate issued for your website.

![pix14](https://user-images.githubusercontent.com/74002629/184856345-02ba08bc-ac02-4ef4-a7a7-47949e29f58a.PNG)

![pix16](https://user-images.githubusercontent.com/74002629/184856437-3d34d4c2-f96a-4707-bc65-09226fb4d47f.PNG)

#### Step 3 - Set up periodical renewal of your SSL/TLS certificate

1. By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently. You can test renewal command in dry-run mode: `sudo certbot renew --dry-run`
2. Best pracice is to have a scheduled job to run renew command periodically. Let us configure a cronjob to run the command twice a day. To do so, edit the crontab file with the following command: `crontab -e`
3. Add following line: `* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`
4. You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.
