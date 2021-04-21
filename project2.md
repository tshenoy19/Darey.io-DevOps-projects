
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

TCP port 22 is open by default on the EC2 machine to access it via SSH. A new rule is added to the EC2 instance to open inbound connection through port 80.

To test if the Nginx server is accessible over the Internet, open a web browser and try to access following url:
```
http://<Public-IP-Address>:80
```

The following page is accessible on the above URL:
![Screen Shot 2021-04-21 at 11 42 30 AM](https://user-images.githubusercontent.com/44268796/115582453-f9857800-a296-11eb-9e9d-f38ac65a5f64.png)

## Step 2 — Installing MySQL

Next step is to install MySQL, a relational database to store and manage data for the site.

```bash
sudo apt install mysql-server
sudo mysql_secure_installation
sudo mysql
```
The installation was successful with the following output:

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

Activate the configuration by linking to the config file from Nginx’s sites-enabled directory:
```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

To disable default Nginx host that is currently configured to listen on port 80, the following command is used:
```
sudo unlink /etc/nginx/sites-enabled/default
```
To reload Nginx server:
```
sudo systemctl reload nginx
```
/var/www/projectLEMP is still empty. An index.html file is created to test:
```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
To check out the server on browser:
```
http://<Public-IP-Address>:80
```
LEMP stack is now fully configured.

## Step 5 – Testing PHP with Nginx
 Open a new file called info.php within your document root in your text editor, this helps to validate if the Nginx server can handoff .php files to the PHP processor :
```
nano /var/www/projectLEMP/info.php
```
```
<?php
phpinfo();
```
To see if it worked, type the below command:
```
http://`server_domain_or_IP`/info.php
```
The webpage should look like this:

![Screen Shot 2021-04-21 at 1 28 06 PM](https://user-images.githubusercontent.com/44268796/115595900-7a4b7080-a2a5-11eb-9836-5a5d7b1e932e.png)

In order to protect sensitive information on the page, after testing it out above, remove the file:
```
sudo rm /var/www/projectLEMP/info.php
```
## Step 6 — Retrieving data from MySQL database with PHP

This is the final step where we create a database, create a table, populate the table with data. 
![Screen Shot 2021-04-21 at 1 35 31 PM](https://user-images.githubusercontent.com/44268796/115599551-c26c9200-a2a9-11eb-832a-b5683625f479.png)

![Screen Shot 2021-04-21 at 1 40 27 PM](https://user-images.githubusercontent.com/44268796/115599636-d4e6cb80-a2a9-11eb-8a14-7eddbd717f55.png)


Then, we create a PHP script (todo_list.php)to connect to the MYSQL database and query for content.

```
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}


```

Finally, when we access the server with the following command - (http://<Public_domain_or_IP>/todo_list.php), the new table will be queried and displayed on the browser. The screenshots of the final step is below:



![Screen Shot 2021-04-21 at 1 53 53 PM](https://user-images.githubusercontent.com/44268796/115599907-1bd4c100-a2aa-11eb-80c9-98146363a813.png)














