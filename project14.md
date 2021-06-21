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







































