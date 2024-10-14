### Prerequisites
- I ensured that two **RHEL8 web servers**, one **MySQL DB server** (Ubuntu 20.04), and one **RHEL8 NFS server** were properly set up. 
- I confirmed Apache was installed and running on both web servers.
- I made sure the directories `/var/www/` from both web servers were mounted to `/mnt/apps/` on the NFS server.


## Configure Apache as Load Balancer

1. I launched an EC2 instance with Ubuntu 20.04 and named it `Project-8-apache-lb`.
2. I opened TCP port 80 by configuring an inbound rule in the security group for the instance.
   
   ![Install Apach](./self_study/images/ec2_instance.png)

3. Install Apache Load Balancer on Project-8-apache-lb Server and Configure It to Point Traffic Coming to LB to Both Web Servers:

```bash
# Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

# Enable the following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

# Restart apache2 service
sudo systemctl restart apache2

# Check if apache2 is up and running
sudo systemctl status apache2
```
![Install Apach](./self_study/images/ll.png)

## Configure Apache as a load balancer

1. I edited the Apache configuration file:
   ```bash
   sudo nano /etc/apache2/sites-available/000-default.conf
   ```
2. I added the following load balancer configuration under `<VirtualHost *:80>`:
   ```bash
   <Proxy "balancer://mycluster">
       BalancerMember http://<WebServer1-Private-IP>:80 loadfactor=5 timeout=1
       BalancerMember http://<WebServer2-Private-IP>:80 loadfactor=5 timeout=1
       ProxySet lbmethod=bytraffic
   </Proxy>

   ProxyPreserveHost On
   ProxyPass / balancer://mycluster/
   ProxyPassReverse / balancer://mycluster/
   ```
   ![Install Apach](./self_study/images/ff.png)
### 4: Verify the load balancer configuration
1. I restarted the Apache service to apply the changes:
   ```bash
   sudo systemctl restart apache2
   ```

2. I checked to ensure Apache was running:
   ```bash
   sudo systemctl status apache2
   ```

3. I accessed the load balancer's public IP or DNS in a browser:
   ```bash
   http://18.232.99.226/index.php
   ```

4. To confirm the load balancing was working, I SSHed into both web servers and checked the access logs:
   ```bash
   sudo tail -f /var/log/httpd/access_log
   ```
   I refreshed the browser multiple times and saw logs being generated on both servers as the requests were distributed.

Note: Test other load balancing methods
- Instead of `bytraffic`, I tested:
   - `byrequests`: balancing based on the number of requests.
   - `bybusyness`: balancing based on the current load of the servers.
   - `heartbeat`: monitoring the server health.

