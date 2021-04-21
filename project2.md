
# WEB STACK IMPLEMENTATION (LEMP STACK)

## Step 0 - Preparing prerequisites

Sign in to AWS free tier account and create a new EC2 Instance of t2.nano family with Ubuntu Server 20.04 LTS (HVM) image.
SSH into the instance.

## Step 1 – Installing the Nginx Web Server
```bash
sudo apt update
sudo apt install nginx
sudo systemctl status nginx
```

We have TCP port 22 open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80.

Test if the Nginx server can respond to requests from the Internet. Open a web browser of your choice and try to access following url:
```
http://<Public-IP-Address>:80
```

If you see following page, then your web server is now correctly installed and accessible through your firewall.
![Screen Shot 2021-04-21 at 11 42 30 AM](https://user-images.githubusercontent.com/44268796/115582453-f9857800-a296-11eb-9e9d-f38ac65a5f64.png)

## Step 2 — Installing MySQL

Next step is to install MySQL, a relational database to store and manage data for the site.

```bash
sudo apt install mysql-server
sudo mysql_secure_installation
sudo mysql
```
If the installation is successful, then the output would be like this:

![Screen Shot 2021-04-21 at 11 49 05 AM](https://user-images.githubusercontent.com/44268796/115583152-a233d780-a297-11eb-9cd5-1f5d0509210c.png)

Exit MySQL:
```bash
mysql> exit
```

## Step 3 – Installing PHP

Next step is to install PHP to process code and generate dynamic content for the web server

```bash
sudo apt install php-fpm php-mysql
```

## Step 4 — Configuring Nginx to Use PHP Processor

Create the root web directory for your_domain as follows:
```bash
sudo mkdir /var/www/projectLEMP
```
Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:
```bash
sudo chown -R $USER:$USER /var/www/projectLEMP
```
Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:
```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```
This will create a new blank file. Paste in the following bare-bones configuration:
```
#/etc/nginx/sites-available/projectLEMP

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
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```

Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:
```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```
This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:
```sudo nginx -t
```
We also need to disable default Nginx host that is currently configured to listen on port 80, for this run:
```
sudo unlink /etc/nginx/sites-enabled/default
```
Reload Nginx to apply the changes:
```
sudo systemctl reload nginx
```
Your new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:
```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
Now go to your browser and try to open your website URL using IP address:
```
http://<Public-IP-Address>:80
```
LEMP stack is now fully configured.

## Step 5 – Testing PHP with Nginx

Next step is to test and validate that Nginx can correctly hand .php files off to the PHP processor.

You can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:
```
nano /var/www/projectLEMP/info.php
```
```
<?php
phpinfo();
```
You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:
```
http://`server_domain_or_IP`/info.php
```















