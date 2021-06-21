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


