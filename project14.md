
### Experience Continuous Integration with Jenkins | Ansible | Artifactory | SonarQube | PHP

#### Set Up

This project is partly a continuation of the Ansible work from previous projects. It will require a lot of servers to simulate all the different environments from dev/ci all the way to production. 

Only create servers based on the current environment that is being worked on at the moment.

The initial focus in this project will be on these three environments:
- CI
- Dev
- Pentest

Both SIT - For System Integration Testing and UAT - User Acceptance Testing do not require a lot of extra installation or configuration. They are basically the webservers holding the applications. But Pentest - For Penetration testing is where security related tests will be conducted, so some other tools and specific configurations will be needed. In some cases, it will also be used for Performance and Load testing. This depends on the company and the team and is a matter of preference. 

In this project, Nginx will serve as a reverse proxy for the sites and tools. Each environment setup is represented in the below table and diagrams.


![Screen Shot 2021-06-21 at 6 52 27 AM](https://user-images.githubusercontent.com/44268796/122751022-3f41cb00-d25d-11eb-842e-c63b405d13c6.png)


![Screen Shot 2021-06-21 at 6 54 02 AM](https://user-images.githubusercontent.com/44268796/122751242-7fa14900-d25d-11eb-8a38-a3a6d897d085.png)


![Screen Shot 2021-06-21 at 6 54 56 AM](https://user-images.githubusercontent.com/44268796/122751316-9c3d8100-d25d-11eb-9fb7-9199496265a0.png)


##### DNS Requirements

Make DNS entries to create a subdomain for each of the environments below. For example, assuming the primary domain name is <domainname.com>, the subdomain for Sonarqube could be sonar.infradev.<domainname.com>.

![Screen Shot 2021-06-21 at 7 01 52 AM](https://user-images.githubusercontent.com/44268796/122752128-909e8a00-d25e-11eb-9c21-9b3888f1fc8b.png)

#### Step 1: Setup


##### The Ansible directory should look like this:

```
├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat
```

![Screen Shot 2021-06-21 at 1 47 44 PM](https://user-images.githubusercontent.com/44268796/122805646-59000400-d297-11eb-8487-ef7037c293e3.png)


##### ci inventory file:
```
[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_repository-Private-IP-Address>
```

![Screen Shot 2021-06-21 at 1 50 29 PM](https://user-images.githubusercontent.com/44268796/122805882-a67c7100-d297-11eb-8aa5-4dce8d18f2a7.png)


##### dev Inventory file
```
[tooling]
<Tooling-Web-Server-Private-IP-Address>

[todo]
<Todo-Web-Server-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[db:vars]
ansible_user=ubuntu
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP-Address>
```
![Screen Shot 2021-06-21 at 1 52 40 PM](https://user-images.githubusercontent.com/44268796/122806137-f5c2a180-d297-11eb-9650-e5af1d021c5e.png)

##### pentest inventory file:
```
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-for-Todo-Private-IP-Address>

[pentest-tooling]
<Pentest-for-Tooling-Private-IP-Address>
```
![Screen Shot 2021-06-21 at 1 53 10 PM](https://user-images.githubusercontent.com/44268796/122806183-06731780-d298-11eb-9e2a-ea8aee208e3a.png)


In the pentest inventory file, a new concept 'pentest:children' has been introduced. This is because the group called 'pentest' covers Ansible execution against both pentest-todo and pentest-tooling simultaneously. But at the same time, this provides flexibility to run specific Ansible tasks against an individual group.

The db group has a slightly different configuration. It uses a RedHat/Centos Linux distro. Others are based on Ubuntu (in this case user is ubuntu). Therefore, the user required for connectivity and path to python interpreter are different.   

With another Ansible concept called group_vars, it is possible to declare and set variables for each group of servers created in the inventory file. For example, if there are variables that are common between both pentest-todo and pentest-tooling, rather than setting these variables in many places, simply use the group_vars for pentest. Since in the inventory file it has been created as pentest:children, Ansible recognizes this and simply applies that variable to both children.

#### Ansible Roles for CI Environment

Add two roles to Ansible:

- [SonarQube](https://www.sonarqube.org)
- [Artifactory](https://jfrog.com/artifactory/)

![Screen Shot 2021-06-21 at 2 50 13 PM](https://user-images.githubusercontent.com/44268796/122812690-ff500780-d29f-11eb-9c6e-f9da6c8a5e0f.png)


SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality, it is used to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities. 

Artifactory is a product by JFrog that serves as a binary repository manager. The binary repository is a natural extension to the source code repository, in that the outcome of the build process is stored. It can be used for certain other automations. But in this project, it will be used to manage our build artifacts.

#### Configuring Ansible For Jenkins Deployment

In previous projects, the Ansible commands were manually launched from a CLI. Now, Ansible will be run from Jenkins UI.

To do this,
- Navigate to Jenkins URL
- Install & Open Blue Ocean Jenkins Plugin
- Create a new pipeline 


![Screen Shot 2021-06-21 at 3 57 58 PM](https://user-images.githubusercontent.com/44268796/122820307-750ca100-d2a9-11eb-9b1c-584fee8e5ef7.png)



##### Select GitHub:

![Screen Shot 2021-06-21 at 3 58 36 PM](https://user-images.githubusercontent.com/44268796/122820414-92da0600-d2a9-11eb-96cf-bb86969d3555.png)



##### Connect Jenkins with GitHub:

![Screen Shot 2021-06-21 at 4 01 07 PM](https://user-images.githubusercontent.com/44268796/122820705-e6e4ea80-d2a9-11eb-9c59-93a79946f56c.png)



##### Login to GitHub & Generate an Access Token:

![Screen Shot 2021-06-21 at 4 01 58 PM](https://user-images.githubusercontent.com/44268796/122820800-0419b900-d2aa-11eb-8786-b9f76db966ec.png)

![Screen Shot 2021-06-21 at 4 03 04 PM](https://user-images.githubusercontent.com/44268796/122820915-2b708600-d2aa-11eb-85f1-36ead7419eaa.png)



##### Copy Access Token, paste it on Jenkins UI and connect:

![Screen Shot 2021-06-21 at 4 04 40 PM](https://user-images.githubusercontent.com/44268796/122821084-65418c80-d2aa-11eb-9e11-c032613a67ce.png)



##### Select ansible-cnfg-mgt from the list of available repositories. Create a new pipeline:

![Screen Shot 2021-06-21 at 4 08 58 PM](https://user-images.githubusercontent.com/44268796/122821651-1cd69e80-d2ab-11eb-80c8-79bfd847623b.png)

At this point, there is no Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give guidance to create one. The Jenkins file will be created manually. click on Administration to exit the Blue Ocean console.

![Screen Shot 2021-06-21 at 4 13 15 PM](https://user-images.githubusercontent.com/44268796/122822054-979fb980-d2ab-11eb-9a7f-b176e2e5161f.png)


The newly created pipeline is now accessible on the Dashboard and takes the name of the repository:

![Screen Shot 2021-06-21 at 4 15 12 PM](https://user-images.githubusercontent.com/44268796/122822284-dd5c8200-d2ab-11eb-879a-e5506ebcc56f.png)

##### Create Jenkinsfile
In the ansible directory, create a folder called 'deploy' and inside the folder, create a file called 'Jenkinsfile'


![Screen Shot 2021-06-22 at 6 54 47 AM](https://user-images.githubusercontent.com/44268796/122912833-bee79c80-d326-11eb-9f29-b0ac9a8cad46.png)

Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build, and the only thing we are doing is using the shell script module to echo Building Stage.

```
pipeline {
    agent any


  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```

Now go back into the Ansible pipeline in Jenkins, and select configure:

![Screen Shot 2021-06-22 at 7 07 52 AM](https://user-images.githubusercontent.com/44268796/122914365-919bee00-d328-11eb-99d2-d18c0836c46b.png)

Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile:

![Screen Shot 2021-06-22 at 7 09 17 AM](https://user-images.githubusercontent.com/44268796/122914579-c5771380-d328-11eb-8d13-e43268629fa4.png)

To make the new branch show up in Jenkins, Jenkins needs to scan the repository. Click on the “Administration” button. Navigate to the Ansible project and click on “Scan repository now". Refresh the page and both branches will start building automatically. Go into Blue Ocean and see both branches there too. 


![Screen Shot 2021-06-22 at 7 16 01 AM](https://user-images.githubusercontent.com/44268796/122915394-b5abff00-d329-11eb-88aa-ec6f11ce4589.png)

Back to the pipeline again, this time click “Build now”:

![Screen Shot 2021-06-22 at 7 23 16 AM](https://user-images.githubusercontent.com/44268796/122916288-b85b2400-d32a-11eb-9773-1368b2377683.png)


This will trigger a build and the effect of our basic Jenkinsfile configuration will be displayed on the console output of the build.

To really appreciate and feel the difference of Cloud Blue UI, trigger the build again from Blue Ocean interface.

1. Click on Blue Ocean 

![Screen Shot 2021-06-22 at 10 21 44 AM](https://user-images.githubusercontent.com/44268796/122941720-a639af80-d343-11eb-8c47-137831fd20a6.png)


2. Select the project and click on the Play button against the branch

![Screen Shot 2021-06-22 at 10 23 10 AM](https://user-images.githubusercontent.com/44268796/122941972-d97c3e80-d343-11eb-9d4d-b768d511e6fc.png)

This pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and it is possible to be able to trigger a build for each branch.

To see this in action:

- Create a new git branch and name it- feature/jenkinspipeline-stages
```
git checkout -b feature/jenkinspipeline-stages
```

- At this point, there is just one stage: Building Stage. Add another stage called Test. Paste the code snippet below and push the new changes to GitHub.
```
pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```

![Screen Shot 2021-06-22 at 10 38 01 AM](https://user-images.githubusercontent.com/44268796/122944559-ee59d180-d345-11eb-915c-49bc119f4890.png)


To make the new branch show up in Jenkins, scan the repository by clicking on the “Administration” button. Navigate to the Ansible project and click on “Scan repository now”. Refresh the page and both branches will start building automatically. Go into Blue Ocean and see both branches there too. 

![Screen Shot 2021-06-22 at 10 43 12 AM](https://user-images.githubusercontent.com/44268796/122945683-bc953a80-d346-11eb-9bc0-b3beb972e7c1.png)


![Screen Shot 2021-06-22 at 10 45 25 AM](https://user-images.githubusercontent.com/44268796/122945970-f6664100-d346-11eb-9779-7a0f4348af93.png)


-  Create a pull request to merge the latest code into the `main branch`
-  After merging the `PR`, go back into your terminal and switch into the `main` branch
-  Pull the latest change


##### Create a complete pipeline by adding new stages to the above configuration:

- Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an `echo` command as in `build` and `test` stages)
   1. Package 
   2. Deploy 
   3. Clean up

- Verify in Blue Ocean that all the stages are working, then merge the feature branch to the main branch
- Eventually, the main branch should have a successful pipeline like this in blue ocean

![Screen Shot 2021-06-22 at 11 30 19 AM](https://user-images.githubusercontent.com/44268796/122954190-3af4db00-d34d-11eb-93fa-7fa12175639c.png)


#### Running Ansible Playbook from Jenkins

After seeing how a typical Jenkins pipeline is created, it is time to create an actual Ansible deployment.






































