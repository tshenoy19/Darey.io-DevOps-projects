### Three-Tier Web Solution with WordPress

The objective of this project is to implement a basic web solution using WordPress by building a storage infrastructure on two Linux servers

#### The two parts of this project comprise of:

 - Creating a Linux-based sub storage system for the web and database layers
 - Installing WordPress and connecting it to a remote MySQL database server

##### Step 1: Launch a web server and build storage volumes

  - Launch an EC2 Instance of RedHat Linux configuration and name it as web-server. 


![Screen Shot 2021-05-10 at 7 34 57 AM](https://user-images.githubusercontent.com/44268796/117653407-596a9280-b162-11eb-8235-807c2403297b.png)


  - Create three 10 GiB EBS volumes in the same AZ and attach them to the EC2 instance which will be the web server. 
  - Attach the three volumes to the EC2 instance 


![Screen Shot 2021-05-10 at 7 35 22 AM](https://user-images.githubusercontent.com/44268796/117653414-5b345600-b162-11eb-8a0b-7ccc81f18011.png)



  
  





  
