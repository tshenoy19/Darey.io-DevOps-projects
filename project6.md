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
































  
  





  
