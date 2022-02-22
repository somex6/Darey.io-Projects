# LOAD BALANCER SOLUTION WITH APACHE
## INTRODUCTION
In this project, I implemented a load balancer solution using Apache which points traffic to two of the webservers in the 3-tier Web Application Architecture that I setup in [project 7](https://github.com/somex6/Darey.io-Projects/blob/main/project7.md).

The following are the steps I took in implementing a load balancer solution:

## Step 0:  Launching The Severs
I launched a new EC2 Instance(Ubuntu 20.04) that will serve as the load balancer and starting the EC2 Instances used in my [project 7](https://github.com/somex6/Darey.io-Projects/blob/main/project7.md). But in this case, out of the 3 EC2 Instances initially launched which are tagged as webservers only 2 were used instead.

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/the%20servers.png)

## Step 1: Configuring Apache As A Load Balancer In The Ubuntu server
1.	Updating and upgrading the server:

`$ sudo apt update`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/apt%20update.png)

`$ sudo apt upgrade`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/apt%20upgrade.png)

2.	Installing Apache:`$ sudo apt install apache2 -y`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/install%20apache2.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/install%20apache2-2.png)

3.	Installing libxml2-dev: `$ sudo apt install libxml2-dev`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/install%20libxml2-dev.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/install%20libxml2-2.png)

4.	Enabling the following modules:
-	`$ sudo a2enmode rewrite`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/a2enmode%20rewrite.png)

-	`$ sudo a2enmode proxy`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/a2enmode%20proxy.png)

-	`$ sudo a2enmode proxy_balancer`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/a2enmode%20proxy_balancer.png)

-	`$ sudo a2enmode proxy_http`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/a2enmode%20proxy_http.png)

-	`$ sudo a2enmode headers`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/a2enmode%20headers.png)

-	`$ sudo a2enmode lbmethod_bytraffic`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/a2enmode%20lbmethod.png)

5.	Restarting Apache:`$ sudo systemctl restart apache2`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/restart%20apache2.png)

6.	To ensure that Apache is running:`$ sudo systemctl status apache2`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/status%20apache2.png)

7.	Opening TCP port 80 on the security group to listen to http request and allow access from any IP address

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/opening%20port%2080%20on%20the%20lb.png)

## Step 2: Configuring The Load Balancer
-	To configure the load balancer, opening the **000-default.conf** file:`$ sudo vi /etc/apache2/sites-available/000-default.conf`
-	Entering the following configuration:
```
        <Proxy "balancer://mycluster">
	               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
	               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
	               ProxySet lbmethod=bytraffic
	               # ProxySet lbmethod=byrequests
	        </Proxy>
	
	        ProxyPreserveHost On
	        ProxyPass / balancer://mycluster/
	        ProxyPassReverse / balancer://mycluster/
```
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/configuring%20the%20load%20balancer.png)

-	Restarting the Apache server:`$ sudo systemctl restart apache2`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/restart%20Apache2-2.png)

## Step 3: Testing The Configuration
-	In order to test the configurations by inspecting the apache log files of the 2 webservers, the ‘/var/log/httpd’ mount made on the webservers to the NFS server on [project 7](https://github.com/somex6/Darey.io-Projects/blob/main/project7.md) was unmounted  in order to make the web servers has its own log directory.
`$ sudo umount /var/log/httpd`

**For webserver A**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/unmounting%20log%20for%20wbsA.png)

**For webserver B**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/unmounting%20logs%20for%20wbsB.png)

-	Executing the following command on the terminal of the two webservers to see how the load balancer works:`$ sudo tail –f /var/log/httpd/access_log`

**For webserver A**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/webA.png)

**For weserver B**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/webB.png)

-	Entering the IP address of the load balancer on my web browser: http://100.24.24.6/index.php

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/testing%20on%20load%20balancer.png)

-	Refreshing the page several times.
-	Viewing the change made in both the terminal:

**For webserver A**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/webA-2.png)

**For webserver B**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/webB-2.png)

## Optional Steps:  Configure Local DNS Names Resolution
-	Editing the hosts file on the Load balancer to assign names(web1 and web2) to the 2 webservers’ private IP addresses: `$ sudo vi /etc/hosts`
-	Adding the following configuration:
```
172.31.85.151 web1
172.31.82.212 web2
```
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/hosts%20file.png)

-	Updating the load balancer config file with those names instead of IP address: `$ sudo vi /etc/apache2/sites-available/000-default.conf`
```
	BalancerMember http://Web1:80 loadfactor=5 timeout=1
	BalancerMember http://Web2:80 loadfactor=5 timeout=1
```
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/load%20balancer%20config.png)

-	Using curl to test the configuration on my load balancer locally: `$ curl http://web1/index.php`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/curl.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project8/curl2.png)
