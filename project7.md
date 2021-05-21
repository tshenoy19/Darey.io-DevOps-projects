
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


#### Step 1: Create NFS Server

##### Create an EC2 instance with RHEL Linux 8 Operating System

##### Configure LVM on the server: 
- Create three logical volumes of 'xfs' format and name them as lv-opt, lv-apps and lv-logs.
- Create three mount points for the logical volumes on the /mnt directory: lv-apps on /mnt/apps for webservers; lv-logs on /mnt/logs for webserver logs and lv-opt on /mnt/opt for jenkins server to be used in the next project



![Screen Shot 2021-05-21 at 9 32 32 AM](https://user-images.githubusercontent.com/44268796/119149107-2f935480-ba1b-11eb-8cbc-369bf5dbf387.png)


![Screen Shot 2021-05-21 at 9 42 52 AM](https://user-images.githubusercontent.com/44268796/119149111-315d1800-ba1b-11eb-9593-152b133a674c.png)


![Screen Shot 2021-05-21 at 9 56 48 AM](https://user-images.githubusercontent.com/44268796/119149115-328e4500-ba1b-11eb-8691-c8e0c0b3af49.png)


##### Next step is to install the NFS server on the EC2 instance and activate it
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

![Screen Shot 2021-05-21 at 10 04 23 AM](https://user-images.githubusercontent.com/44268796/119149917-f4455580-ba1b-11eb-8b53-b8e8f1b89e23.png)










