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


#### Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks
 
 Here we configure a Jenkins Build that will be triggered by GitHub webhooks. The Build will execute a task to retrieve code from the GitHub repository and store it locally on the Jenkins server. 
 
First step is to enable webhooks in the GitHub repository settings: 

![Screen Shot 2021-05-31 at 2 59 43 PM](https://user-images.githubusercontent.com/44268796/120315514-72bab680-c2aa-11eb-8ecf-094434a49fbb.png)


Next, on the Jenkins web console, click “New Item” and create a “Freestyle project” and name it "tooling_github"

![Screen Shot 2021-05-31 at 3 00 46 PM](https://user-images.githubusercontent.com/44268796/120315725-a8f83600-c2aa-11eb-8450-62ae2cf0f3a5.png)


Configure the project to connect with your GitHub repository by entering the URL and credentials. Save the configuration and click on the "Build Now" link. 


![Screen Shot 2021-05-31 at 3 06 19 PM](https://user-images.githubusercontent.com/44268796/120315874-ce853f80-c2aa-11eb-83d9-09d0e42185ca.png)



![Screen Shot 2021-05-31 at 3 06 51 PM](https://user-images.githubusercontent.com/44268796/120315886-d1803000-c2aa-11eb-8ca1-f534e6f1fdd8.png)


The Workspace for the first job has retrieved all the files/directories from the GitHub repository and stored it locally.

![Screen Shot 2021-06-01 at 7 31 19 AM](https://user-images.githubusercontent.com/44268796/120316395-5ec38480-c2ab-11eb-88f2-4ddc9ec59f78.png)


This build is triggered only when activated manually. 

###### Adding additional configurations to the project "tooling_github"

Click “Configure” your job/project and add these two configurations: 

Configure triggering the job from GitHub webhook:

![Screen Shot 2021-06-01 at 9 06 38 AM](https://user-images.githubusercontent.com/44268796/120328290-ae5c7d00-c2b8-11eb-9d8f-c705b4a6ea82.png)


Configure “Post-build Actions” to archive all the files - files resulted from a build are called “artifacts”

![Screen Shot 2021-06-01 at 9 09 43 AM](https://user-images.githubusercontent.com/44268796/120328729-1ca13f80-c2b9-11eb-83fb-cf551dc490ac.png)


The changes to the GitHub master branch automatically trigger a new build on the Jenkins server and the artifacts are updated and archived. The updated artifacts can also be accessible at ``` ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/ ```

![Screen Shot 2021-06-01 at 9 55 29 AM](https://user-images.githubusercontent.com/44268796/120335485-80c70200-c2bf-11eb-827e-fc9cd3fc319d.png)



#### Step 3 - Configure Jenkins to copy files to NFS server via SSH

The next step is to copy the artifacts to the NFS server to /mnt/apps directory. For this, a plugin called "Publish over SSH" can be used. 

###### Install “Publish Over SSH” plugin 
On main dashboard select “Manage Jenkins” and choose “Manage Plugins” menu item.

On “Available” tab search for “Publish Over SSH” plugin and install it:

###### Configure the job/project to copy artifacts over to NFS server

On main dashboard select “Manage Jenkins” and choose “Configure System” menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
Arbitrary name
Hostname - can be private IP address of your NFS server
Username - ec2-user (since NFS server is based on EC2 with RHEL 8)
Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.


###### Save the configuration, open your Jenkins job/project configuration page and add another one “Post-build Action”

Configure it to send all files produced by the build into the previously defined remote directory. In our case we want to copy all files and directories - so we use **. 

![Screen Shot 2021-06-01 at 10 26 21 AM](https://user-images.githubusercontent.com/44268796/120340237-d1405e80-c2c3-11eb-88a5-af10a938793c.png)

















