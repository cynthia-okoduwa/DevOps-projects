## WEB SOLUTION WITH WORDPRESS
### Part 1: Configure storage subsystem for Web and Database servers based on Linux OS.
#### Steps
1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
2. Attach all three volumes one by one to your Web Server EC2 instance
3. Open up the Linux terminal to begin configuration of the instance. Use `lsblk` command to inspect what block devices are attached to the server. The 3 newly
created block devices are names **xvdf, xvdh, xvdg**
4. 
