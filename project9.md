#### Tooling Website deployment automation with Continuous Integration. Introduction to Jenkins

This project further builds upon Project 8 by incorporating one of the most popular CI/CD tools -Jenkins. In this project, Jenkins is used to make sure that every change made to the GitHub source code will be automatically updated to the Tooling website.

#### Task: Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server

The architecture of the implementation is below:

![Screen Shot 2021-05-31 at 2 26 41 PM](https://user-images.githubusercontent.com/44268796/120229948-46f0ef80-c21c-11eb-9581-1eaee63823b8.png)


#### Step 1 - Install Jenkins server

Create an EC2 instance of type Ubuntu 20.04 and name it as 'jenkins-server'. As Jenkins is a Java-based application, install JDK.

![Screen Shot 2021-05-31 at 2 31 46 PM](https://user-images.githubusercontent.com/44268796/120230264-ef9f4f00-c21c-11eb-84ed-3c32e71a5bc5.png)


```
sudo apt update
sudo apt install default-jdk-headless
```
Install Jenkins:
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
Ensure that Jenkins is up and running:
```
sudo systemctl status jenkins
```

![Screen Shot 2021-05-31 at 2 39 33 PM](https://user-images.githubusercontent.com/44268796/120230768-05f9da80-c21e-11eb-83bf-82f782efded0.png)


Allow traffic inbound on Port 8080 on the Jenkins server. Jenkins is now accessible on the browser at http://(Jenkins-Server-Public-IP-Address-or-Public-DNS-Name):8080
    
![Screen Shot 2021-05-31 at 2 42 19 PM](https://user-images.githubusercontent.com/44268796/120230977-69840800-c21e-11eb-8861-bc203d5aa0a2.png)
    
 To retrieve the default admin password:
 ```
 sudo cat /var/lib/jenkins/secrets/initialAdminPassword
 ```
 
  Install selected plugins. 
 
 ![Screen Shot 2021-05-31 at 2 46 46 PM](https://user-images.githubusercontent.com/44268796/120231262-09419600-c21f-11eb-8bac-d23d036beab5.png)
 
After the plugin installations, create an admin user. Jenkins will then provide a URL.

![Screen Shot 2021-05-31 at 2 49 36 PM](https://user-images.githubusercontent.com/44268796/120231450-6dfcf080-c21f-11eb-9dda-6a590e7d2df9.png)


The Jenkins setup is now complete.

![Screen Shot 2021-05-31 at 2 51 41 PM](https://user-images.githubusercontent.com/44268796/120231594-b87e6d00-c21f-11eb-8f08-9d55feea32ae.png)











