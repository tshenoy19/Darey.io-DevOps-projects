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

SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality, it is used to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities. 

Artifactory is a product by JFrog that serves as a binary repository manager. The binary repository is a natural extension to the source code repository, in that the outcome of the build process is stored. It can be used for certain other automations. But in this project, it will be used to manage our build artifacts.













































