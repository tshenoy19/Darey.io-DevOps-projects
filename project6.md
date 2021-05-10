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



































  
  





  
