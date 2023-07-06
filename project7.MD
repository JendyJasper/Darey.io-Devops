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

