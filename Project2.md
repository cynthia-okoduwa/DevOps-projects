## WEB STACK IMPLEMENTATION (LEMP STACK)
### STEP 1 – INSTALLING THE NGINX WEB SERVER
#### Steps
1. To start, update server’s package index. Afterwards, use apt install to get the Nginx installation going. To start the update run: **Sudo apt update**
2. Next, install Nginx by running: **sudo apt install nginx**
3. At the prompt, enter Y to confirm that you want to install Nginx. This would complete the installation process.
4. To confirm that nginx was successfully installed and is running as a service in Ubuntu, run: **sudo systemctl status nginx**
5. Green indicator shows that the server was successfully install and running.
6. The server is running and we can access it locally and from the Internet but to access it from the internet port 80 must be open to allow traffic from the internet in.
7. To test that the server can be accessed locally from the instance run the curl command: **curl http://localhost:80** The output shows that the server is accessible from the local host.
8. To test that the server can be accessed from the internet, open a browser and type the following url with the public IP of your Ubuntu instance; 
**http://<Public-IP-Address>:80**
