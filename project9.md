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









