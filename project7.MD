# DEVOPS TOOLING WEBSITE SOLUTION

> We want to introduce a set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects.
> The tools we want our team to be able to use are well known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of:

- Jenkins – free and open source automation server used to build CI/CD pipelines.
- Kubernetes – an open-source container-orchestration system for automating computer application deployment, scaling, and management.
- Jfrog Artifactory – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
- Rancher – an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.
- Grafana – a multi-platform open source analytics and interactive visualization web application.
- Prometheus – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
- Kibana – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.

#### Create 3 Webservers and 1 storage server. Use RHEL-8.6.0_HVM-20220503-x86_64-2-Hourly2-GP2 AMI for RHEL 8.
<img width="1437" alt="Screenshot 2023-06-27 at 4 28 36 PM" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/7e73ba5a-d05f-4d36-812e-99c9f28be1a7">

## Prepare NFS Server

- Configure LVM on the Server.
- Format them as xfs
- Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs
- Create mount points on /mnt directory for the logical volumes as follow:
  - Mount lv-apps on /mnt/apps – To be used by webservers
  - Mount lv-logs on /mnt/logs – To be used by webserver logs
  - Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

- Create 3 EBS volumes. Make sure the volumes are in the same availability zone as your ec2 instances. Attach the 3 EBS volumes to the NFS server
<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/68d78f4b-0e62-42f4-b518-88d550359d23">
- Login to the web server ec2 instance to begin configurations
- Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices as shown in the screenshot above. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.
<img width="607" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/fa3ea19b-f3a5-45d9-8b78-d6d3fdb112f0">

- You could see the 3 volumes attached to this instance.
- Use gdisk utility to create a single partition on each of the 3 disks: `sudo gdisk /dev/xvdf` . Repeat the process for the other two attached volumes. Make sure you select the Linux file system type which has a hex code of 8300.
<img width="607" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/ec3bb0d1-2722-4ed4-be11-941c23fa732e">

- You can see that the 3 partitions are all created.
- Install lvm2 package using `sudo yum install lvm2` . Run `sudo lvmdiskscan` command to check for available partitions.
- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM. You can create the one at a time or all at once. See the picture below. Run `sudo pvs` to make sure the physical volumes are created.
<img width="1036" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/8eb4c9b1-ac3f-4b35-9096-2721011be6fd">

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG nfsStorage-vg: `sudo vgcreate nfsStorage-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
- Verify that your VG has been created successfully by running `sudo vgs`
<img width="1204" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/2d65fc31-c9ef-4a5f-977b-4541d8535520">

- Use lvcreate utility to create 3 logical volumes: `lv-opt lv-apps, and lv-logs`: `sudo lvcreate -n lv-opt -L 9G nfsStorage-vg`, `sudo lvcreate -n lv-apps -L 9G nfsStorage-vg`, and `sudo lvcreate -n lv-logs -L 9G nfsStorage-vg`
- Verify that your Logical Volume has been created successfully by running sudo lvs
<img width="1408" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/078df0ae-bd37-4417-b9a9-51fd6412e34f">

- Verify all are setup: `sudo lsblk`
<img width="1008" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/5ad91a44-3cff-42f4-91b6-1576703d2f23">

- Use mkfs.xfs to format the logical volumes with xfs filesystem: `sudo mkfs -t xfs /dev/nfsStorage-vg/lv-apps` and `sudo mkfs -t xfs /dev/nfsStorage-vg/lv-logs`, and `sudo mkfs -t xfs /dev/nfsStorage-vg/lv-opt`

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/f94df682-e950-48be-a01b-5a3c6d519c05">

- Create 3 directories which will be used as the mount points: `sudo mkdir /mnt/apps`, `sudo mkdir /mnt/logs` and `sudo mkdir /mnt/opt`
- Create mount points on /mnt directory for the logical volumes as follow:
  - Mount lv-apps on /mnt/apps – To be used by webservers
  - Mount lv-logs on /mnt/logs – To be used by webserver logs
  - Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8
- Mount /mnt/apps on lv-apps logical volume: `sudo mount /dev/nfsStorage-vg/lv-apps /mnt/apps`
- Mount /mnt/logs on lv-logs logical volume: `sudo mount /dev/nfsStorage-vg/lv-logs /mnt/logs`
- Mount /mnt/opt on lv-opt logical volume: `sudo mount /dev/nfsStorage-vg/lv-opt /mnt/opt`

#### Install NFS server, configure it to start on reboot and make sure it is u and running
- `sudo yum -y update`
- `sudo yum install nfs-utils -y`
- `sudo systemctl start nfs-server.service`
- `sudo systemctl enable nfs-server.service`
- `sudo systemctl status nfs-server.service`

#### Export the mounts for webservers’ subnet cidr to connect as clients. To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:
#### set up permission that will allow our Web servers to read, write and execute files on NFS:
- `sudo chown -R nobody: /mnt/apps`
- `sudo chown -R nobody: /mnt/logs`
- `sudo chown -R nobody: /mnt/opt`

- `sudo chmod -R 777 /mnt/apps`
- `sudo chmod -R 777 /mnt/logs`
- `sudo chmod -R 777 /mnt/opt`

- `sudo systemctl restart nfs-server.service`
<img width="1205" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/44c633ce-9c64-42f2-85b2-f851fe097ebb">

#### Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):
```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv
```
<img width="851" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/7b490b09-d2d2-4b4e-8826-176503516936">

#### Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)
`rpcinfo -p | grep nfs`

<img width="851" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/dd823812-e1c4-4bb9-b365-ff3ec1a0acc6">

> Important note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049

<img width="1040" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/51a624f6-7781-4cb9-af62-bc6bb895eb19">


## PREPARE DATABASE SERVER
- `sudo apt update`
- Install mysql server: `sudo apt install mysql-server`
- Verify that the service is up and running by using `sudo systemctl status mysql`, if it is not running, restart the service and enable it so it will be running even after reboot: `sudo systemctl restart mysql` and `sudo systemctl enable mysql`
- Create a database and name it tooling: `create database tooling`
- Create a database user and name it webaccess: 
```
CREATE USER `webaccess`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'password';
```
- Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr: `GRANT ALL ON tooling.* TO 'webaccess'@'<Web-Server-Private-IP-Address>';`
- `flush privileges`

## PREPARE THE WEB SERVER
> We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database.We already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

> This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

During the next steps we will do following:
 - Configure NFS client (this step must be done on all three servers)
 - Deploy a Tooling application to our Web Servers into a shared NFS folder
 - Configure the Web Servers to work with a single MySQL database

- Launch a new EC2 instance with RHEL 8 Operating System
- Install NFS Client: `sudo yum install nfs-utils nfs4-acl-tools -y`
- Mount /var/www/ and target the NFS server’s export for apps: 
```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```

- Verify that NFS was mounted successfully by running `df -h`. Make sure that the changes will persist on Web Server after reboot: `sudo vi /etc/fstab`
- add following line
```
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
```

<img width="1040" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/217a8f6b-7d82-47cb-9af5-636a27759f5b">

- Install Remi’s repository, Apache and PHP:
```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1
```

### Repeat same for 2 other web servers

- Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. make sure the mount point will persist after reboot.: `sudo mount -t nfs -o rw,nosuid 172.31.84.168:/mnt/logs /var/log/httpd/` where the IP address is the NFS storage private IP
- fork and clone this repo: https://github.com/darey-io/tooling
- Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html - :`sudo cp -R tooling/html/. /var/www/html`


> Note 1: Do not forget to open TCP port 80 on the Web Server.

> Note 2: If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux `sudo setenforce 0`
To make this change permanent – open following config file sudo vi /etc/sysconfig/selinux and set SELINUX=disabled then restart httpd.

<img width="1440" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/6ede6cea-fa76-49a2-96f9-ffb5bc20732b">

- Update the website’s configuration to connect to the database (in /var/www/html/functions.php file). Apply tooling-db.sql script to your database using this command `mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`
