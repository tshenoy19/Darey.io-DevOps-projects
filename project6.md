### Three-Tier Web Solution with WordPress

The objective of this project is to implement a basic web solution using WordPress by building a storage infrastructure on two Linux servers

#### The two parts of this project comprise of:

 - Creating a Linux-based sub storage system for the web and database layers
 - Installing WordPress and connecting it to a remote MySQL database server

##### Step 1: Launch a web server and build storage volumes

  ###### Launch an EC2 Instance of RedHat Linux configuration and name it as web-server. 


![Screen Shot 2021-05-10 at 7 34 57 AM](https://user-images.githubusercontent.com/44268796/117653407-596a9280-b162-11eb-8235-807c2403297b.png)


 ###### Create three 10 GiB EBS volumes in the same AZ and attach them to the EC2 instance which will be the web server. 
 ###### Attach the three volumes to the EC2 instance 


![Screen Shot 2021-05-10 at 7 35 22 AM](https://user-images.githubusercontent.com/44268796/117653414-5b345600-b162-11eb-8a0b-7ccc81f18011.png)


###### Connect to the instance and inspect the volumes attached to it with the ``` lsblk ``` command


![Screen Shot 2021-05-10 at 10 24 08 AM](https://user-images.githubusercontent.com/44268796/117674841-06043e80-b17a-11eb-9c42-f4f6eb1a62f8.png)


###### After checking the amount of diskspace available on each volume with the ``` gdisk ``` command, create a single partition in each of the three volumes

```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh
```


![Screen Shot 2021-05-10 at 10 47 47 AM](https://user-images.githubusercontent.com/44268796/117678212-2aade580-b17d-11eb-918a-0672545b69a2.png)


![Screen Shot 2021-05-10 at 10 50 51 AM](https://user-images.githubusercontent.com/44268796/117678653-97c17b00-b17d-11eb-93b7-31ffadc413b0.png)


###### Install lvm2 package
```
sudo yum install lvm2
```
###### Check for available partitions
```
sudo lvmdiskscan 
```
###### With the ``` pvcreate ``` command, mark each of the three disks as physical volumes to be used by LVM
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
###### Check if the physical volumes have been created
```
sudo pvs
```

![Screen Shot 2021-05-10 at 10 52 49 AM](https://user-images.githubusercontent.com/44268796/117678956-de16da00-b17d-11eb-929b-c08766e67d3d.png)

###### With the ``` vgcreate ``` command, create a volume group of the three physical volumes and name is 'webdata-vg'

```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```

###### Check if VG has been created
```
sudo vgs
```
![Screen Shot 2021-05-10 at 10 56 40 AM](https://user-images.githubusercontent.com/44268796/117679583-67c6a780-b17e-11eb-9a6b-0144a34ff664.png)


###### With ``` lvcreate ``` command, create two separate logical volumes, app-lv (to store website data) and logs-lv (to store logs data) and verify the configuration
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
sudo lvs
```

![Screen Shot 2021-05-10 at 11 22 27 AM](https://user-images.githubusercontent.com/44268796/117683387-030d4c00-b182-11eb-84ef-4b602cfe1b9f.png)


###### Now check to see the entire storage setup
```
sudo vgdisplay -v
sudo lsblk
```

![Screen Shot 2021-05-10 at 11 24 01 AM](https://user-images.githubusercontent.com/44268796/117683594-3b148f00-b182-11eb-82fc-ff3fbaf76710.png)

###### Format the logical volumes with ext4 filesystem
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
###### Create two separate directories to store website files and log data
```
sudo mkdir -p /var/www/html
sudo mkdir -p /home/recovery/logs
```
###### Mount the  /var/www/html on apps-lv logical volume
```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
###### With ``` rsync ``` command, backup all the files in the log directory /var/log into /home/recovery/logs
```
sudo rsync -av /var/log/. /home/recovery/logs/
```
###### Now, mount the /var/logs file onto the data-vg logical volume
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```
###### Restore the data on the /var/logs file as all the existing data is deleted 
```
sudo rsync -av /home/recovery/logs/log/. /var/log
```
###### To make the mount configuration persist, make additions to the /etc/fstab file as seen below with the UUIDs:
```
sudo blkid
```
![Screen Shot 2021-05-10 at 1 50 14 PM](https://user-images.githubusercontent.com/44268796/117702557-a8321f80-b196-11eb-9790-51e3b0f2ebff.png)


![Screen Shot 2021-05-10 at 1 59 08 PM](https://user-images.githubusercontent.com/44268796/117703673-eda31c80-b197-11eb-829b-edf3338e7566.png)


###### To test the configuration and reload the daemon:
```
sudo mount -a
sudo systemctl daemon-reload
```

###### To view and check the final setup, use:
```
df -h
```

![Screen Shot 2021-05-10 at 2 03 57 PM](https://user-images.githubusercontent.com/44268796/117704204-9487b880-b198-11eb-8a57-357b108c55e0.png)

##### Step 2: Create the DB Server

###### Launch a Red Hat EC2 machine and name it db-server. Follow all the steps similar to web-server above. Name the logical volume as 'db-lv' instead of 'app-lv' and it should be mounted on /db directory instead of the /www/var/html directory used previously for web-server

##### Step 3: Installation of WordPress on web-server

###### Update the repository, then install wget, Apache and dependencies
```
sudo yum -y update
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```
###### Start Apache and also install PHP with its dependencies
```
sudo systemctl enable httpd
sudo systemctl start httpd
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
```
###### Restart Apache
```
sudo systemctl restart httpd
```
###### Install WordPress and copy it to /var/www/html
```
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -R wordpress /var/www/html/
```
###### Configure Selinux
```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```

##### Step 4: Installation of MySQL on db-server

```
sudo yum update
sudo yum install mysql-server
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```

##### Step 5: Configuration of db-server to work with WordPress on web-server
```
sudo mysql
```
```mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

![Screen Shot 2021-05-10 at 4 24 57 PM](https://user-images.githubusercontent.com/44268796/117720119-454b8300-b1ac-11eb-88b5-a43bfb1c2f4a.png)




##### Step 6: Configuration of the WordPress connection to db-server database

###### First, configure the security group on db-server to allow traffic on port 3302 inbound from only the IP address of web-server

###### Install MySQL client on web-server and test the connection to the remote db-server
```
sudo yum install mysql-server
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```

###### Check to see if the connection has been established and the databases can be viewed from web-server

###### Next, configure the security group on web-server to allow TCP traffic on Port 80 from everywhere 0.0.0.0/0

###### The WordPress website can be accessed on http://<Web-Server-Public-IP-Address>/wordpress/
 
 







































  
  





  
