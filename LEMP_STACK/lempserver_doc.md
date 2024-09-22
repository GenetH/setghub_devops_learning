# LEMP Stack Setup on Ubuntu 24.04 LTS

This guide describes the step-by-step process I followed to set up a LAMP stack on an Ubuntu server. This environment, which includes Linux, Nginx, MySQL, and PHP , is essential for hosting websites and web applications.

## Step 0: Server Setup

1. **Launch EC2 Instance:**
   I launched an EC2 instance of type **t2.micro** running **Ubuntu 24.04 LTS (HVM)** in the **us-east-1** region using the AWS console.

   ![Launch Instance](./self_study/images/ec2_create.png)
   ![Launch Instance](./self_study/images/ec2_instance.png)

2. **Configure Security Group:**
 
   To allow necessary traffic, I set up security rules to open HTTP (Port 80) to the world for web traffic and restrict SSH (Port 22) access to my IP for security reasons.

   ![Security Rules](./self_study/images/security-rule.png)

3. **Create SSH Key Pair:**
   I created an SSH key pair named my-key for secure access, adjusted file permissions, and connected to the instance using the following commands:
     ```bash
     chmod 400 my-key.pem
     ssh -i my-key.pem ubuntu@3.80.233.195
     ```
    ![Connect to instance](./self_study/images/connect_instance.png)

## Step 1: Installing the Nginx Web Server

1. **Update the package list**

   I started by updating my server’s package index to ensure I had access to the latest versions. I ran:
   ```bash
   sudo apt update
   ```
   ![Update Package](./self_study/images/update_package.png)
2. **Install Nginx**

   ```bash
   sudo apt install nginx
   ```
   I confirmed the installation when prompted by typing "Y."
   ![Nginx installed](./self_study/images/nginx_installed.png)
   
3. **Verify Nginx is running**

   Once Nginx was installed, I checked its status to ensure it was running by using:
   ```bash
   sudo systemctl status nginx
   ```
   ![Nginx status](./self_study/images/nginx_running.png)
   The service showed as "active" in green, meaning Nginx was up and running.

4. **Test Nginx locally**

   To check if Nginx was running properly, I tested it locally on the server by running:
   ```bash
   curl http://localhost:80
   ```
   I also tried:
   ```bash
   curl http://127.0.0.1:80
   ```
   ![Nginx installed](./self_study/images/nginx_check.png)
   Both commands confirmed that Nginx was serving content by returning the welcome page.

5. **Access Nginx via public IP**

   Finally, I tested the setup by accessing the Nginx welcome page from my web browser. I opened my browser and went to:
   ```
   http://3.80.233.195:80
   ```
   ![Nginx status](./self_study/images/nginx_status.png)
   I saw the Nginx welcome page, confirming that Nginx was successfully installed and accessible over the internet.

## Step 2: Installing MYSQL

1. **Install MySQL**

   To get MySQL installed, I used the following command:
   ```bash
   sudo apt install mysql-server
   ```
   When prompted, I confirmed the installation by typing "Y" and hitting `ENTER`.
   ![Mysql install](./self_study/images/istall_mysql.png)

2. **Log into MySQL**

   Once the installation was done, I logged into MySQL as the root user by typing:
   ```bash
   sudo mysql
   ```
   ![Mysql access](./self_study/images/mysql_access.png)
   This allowed me to access the MySQL monitor with administrative privileges.

3. **Set Root User Authentication and Password**
   ```
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
   ```
   This command changed the root user's authentication method and set the password to PassWord.1.
4. **Secure MySQL**

   To improve security, I exited MySQL and ran the following script:
   ```bash
   sudo mysql_secure_installation
   ```
   ![Mysql make sercure](./self_study/images/secure_db.png)
   I ran the following command to start the MySQL secure installation script and I made the appropriate selections based on my security needs. After answering the remaining prompts (such as removing anonymous users, disallowing remote root login, and removing the test database), MySQL was fully secured.

5. **Log in with Root Password**

   After securing MySQL, I tested the root password by running:
   ```bash
    sudo mysql -p
   ```
   ![Mysql access](./self_study/images/db_access.png)
   This prompted me for the password I had set during the security configuration, confirming everything was properly set up.


## Step 3: Install PHP

   With Nginx and MySQL installed, I moved on to installing PHP to handle dynamic content and communicate with MySQL databases.

   To install PHP along with the necessary components for Nginx, I ran:
   ```bash
   sudo apt install php-fpm php-mysql
   ```
   When prompted, I confirmed the installation by typing "Y" and pressing `ENTER`.
   ![php install](./self_study/images/php_install.png)

   This installed **php-fpm** (PHP FastCGI Process Manager) to allow Nginx to process PHP requests, and **php-mysql**, which enables PHP to communicate with MySQL databases.

## Step 4: Configuring Nginx to Use the PHP Processor

1. **Create the Root Web Directory**
   I started by creating the root directory for my project:
   ```bash
   sudo mkdir /var/www/projectLEMP
   ```

2. **Change Ownership of the Directory**
   Next, I made sure my user owned the directory by running:
   ```bash
   sudo chown -R $USER:$USER /var/www/projectLEMP
   ```

3. **Open the Nginx Configuration File**
   I opened a new configuration file in Nginx’s `sites-available` directory. For this, I used `nano` as my editor:
   ```bash
   sudo nano /etc/nginx/sites-available/projectLEMP
   ```

4. **Configure Nginx for PHP Processing**
   I pasted the following configuration into the blank file:
   ```nginx
   server {
       listen 80;
       server_name projectLEMP www.projectLEMP;
       root /var/www/projectLEMP;

       index index.html index.htm index.php;

       location / {
           try_files $uri $uri/ =404;
       }

       location ~ \.php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
       }

       location ~ /\.ht {
           deny all;
       }
   }
   ```

   This configuration tells Nginx to serve files from `/var/www/projectLEMP` and process `.php` files using PHP-FPM.

5. **Save and Exit `nano`**:
   After entering the configuration, I saved and exited `nano` by pressing `CTRL + X`, then `Y` to confirm, and `ENTER` to save.

6. **Enable the New Nginx Configuration**:
   I created a symbolic link to enable the configuration file:
   ```bash
   sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
   ```

7. **Test Nginx Configuration for Syntax Errors**:
   Before reloading Nginx, I tested the configuration to ensure there were no errors:
   ```bash
   sudo nginx -t
   ```

   If everything was correct, I saw this message:
   ```bash
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successful
   ```

8. **Reload Nginx to Apply Changes**:
   I reloaded Nginx to apply the new configuration:
   ```bash
   sudo systemctl reload nginx
   ```

9. **Create a Test HTML File**:
   I created a simple test file to confirm that my Nginx server was working:
   ```bash
   echo 'Hello LEMP from hostname' > /var/www/projectLEMP/index.html
   ```

10. **Test the Setup**:
    I opened my web browser and navigated to the server’s public IP address:
    ```
    http://<Public-IP-Address>:80
    ```

    If everything worked correctly, I saw the text:
    ```
    Hello LEMP from hostname
    ```
   By following these steps, I successfully configured Nginx to process PHP and served content from my project directory.

