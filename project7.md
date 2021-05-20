
### DevOps Tooling Website Solution

The website will display a dashboard of some of the most useful DevOps tools as listed below:
- Jenkins
- Kubernetes
- JFrog Artifactory
- Rancher
- Docker
- Grafana
- Prometheus
- Kibana

The project will comprise of the following resources:
- AWS as infrastructure
- Web Server with Red Hat Enterprise Linux 8
- DB Server with Ubuntu 20.04 and MySQL
- Storage solution: Red Hat Enterprise Linux 8 + NFS Server


#### Step 1: Set up NFS Server

##### Create an EC2 instance with RHEL Linux 8 Operating System

##### Configure LVM on the server: 
###### Create three logical volumes of 'xfs' format and name them as lv-opt, lv-apps and lv-logs.
###### Create three mount points for the logical volumes on the /mnt directory: lv-apps on /mnt/apps for webservers; lv-logs on /mnt/logs for webserver logs and lv-opt on /mnt/opt for jenkins server to be used in the next project











