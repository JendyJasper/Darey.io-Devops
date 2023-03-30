# WEB SOLUTION WITH WORDPRESS

> In this project, we are going to work on deploying a web solution with Wordpress. We are going to use Redhat. I used this RHEL image ``` RHEL-8.2.0_HVM-20210907-x86_64-0-Hourly2-GP2```.
> We will have 2 different EC2 instances, one named ```web server``` which is where we will install and configure our wordpress, php, mysql client and apache.
> we will also set up 3 AWS EBS volumes for the "web server" ec2 instance, attach them to the instance, partition them using gdisk(it will be linux LVM type), create physical volumes, create volume group, 
> create logical volumes all using the 3 EBS volumes. Then we will mount the logical volumes to 2 different directories to handle the web server - ```/var/www/html``` and to handle the logs - ```/var/log```. 
> We will back up the contents inside ```/var/log``` into ```/home/recovery/logs``` so we don't lose the existing logs. When we are done mounting to ```/var/log```, we can then move the backed up logs back into ```/var/log```.
> We will create ext4 file systems using ```mkfs.ext4``` command from the disk partitions and edit the ```/etc/fstab``` file so it will auto mount upon reboot. Then we will use ```systemctl``` command to restart the services.
> The other ec2 instance is named ```db server```. This instance will handle all our database data. We will install mysql server on this instance. The ```web server``` instance should be able to communicate with the
> ```db server``` using mysql instead of ssh. It means we will also set up a database name, create a user, create a password, grant privilleges to the created user, and pass the login credentials
> to the ```web server``` ec2 instance so the web server can use it to communicate with the database server. We will also repeat the process of creating and attaching a volume as we did for the web server instance. The Only difference 
> is that we will create only one logical volume, and mount it to ```/db``` for the ```db server```. 




______________________________________________________________________________________


- Create two ec2 instances, name the first one ```web server``` and the second one ```db server```
- <img width="1410" alt="image" src="https://user-images.githubusercontent.com/29708657/228852470-2ead3de0-861a-4567-afb8-aa1f7aca7301.png">

### Configure the web server ec2 instance

- Create 3 EBS volumes. Make sure the volumes are in the same availability zone as your ec2 instances
- <img width="1435" alt="image" src="https://user-images.githubusercontent.com/29708657/228854394-302da274-d12b-4c67-a97d-e1ef89bf05b9.png">
- Attach all the three volumes to the ```web server``` ec2 instance
- <img width="1440" alt="image" src="https://user-images.githubusercontent.com/29708657/228855681-bb8646d8-0740-4f90-afb1-46ff0613cf89.png">
- Login to the web server ec2 instance to begin configurations
- Use ```lsblk``` command to inspect what block devices are attached to the server. Notice names of your newly created devices as shown in the screenshot above. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.
- <img width="714" alt="image" src="https://user-images.githubusercontent.com/29708657/228861352-b6e0bb59-0aa7-41fd-b572-6d18c16f7d40.png">- 
- In the above picture, you will see three disks namely xvdf, xvdg, and xvdh. These are the 3 volumes we attached to this server. Each of them has 10GB of storage capacity.
- Use ```df -h``` command to see all mounts and free space on your server. We can't see anything for the new disks yet because we haven't partiotioned and mounted them yet.
- Use gdisk utility to create a single partition on each of the 3 disks: ```sudo gdisk /dev/xvdf```. Repeat the process for the other two attached volumes. Make sure you select the Linux LVM type which has a hex code of 8e00. THis is neccessary as we will create logical volumes with these partitions
- <img width="717" alt="image" src="https://user-images.githubusercontent.com/29708657/228868806-5d5ac521-2caa-429b-99ae-e02072cd94fc.png">
- After creating the partitions, run the ```lsblk``` command again to verify the partitions creations.
- <img width="698" alt="image" src="https://user-images.githubusercontent.com/29708657/228871063-9d45cf71-30fb-4b73-b0db-736b622e5f19.png">
- Install lvm2 package using ```sudo yum install lvm2```. Run ```sudo lvmdiskscan``` command to check for available partitions.
- <img width="511" alt="image" src="https://user-images.githubusercontent.com/29708657/228910736-20438b2a-4589-4a84-810f-d0ca078fca77.png">
- Use ```pvcreate``` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM. You can create the one at a tine or all at once. See the picture below
- <img width="782" alt="image" src="https://user-images.githubusercontent.com/29708657/228911529-d54c126a-64e9-4014-a8f3-f305c52c0a47.png">
- Verify that your Physical volume has been created successfully by running ```sudo pvs```
- <img width="512" alt="image" src="https://user-images.githubusercontent.com/29708657/228913177-82382dd9-c06d-4ce0-a79e-b8d464b59fd7.png">
- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg: ```sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1```
- Verify that your VG has been created successfully by running ```sudo vgs```
- <img width="985" alt="image" src="https://user-images.githubusercontent.com/29708657/228913994-34830203-b88b-405d-bb55-1e0e2db82f55.png">
- Use lvcreate utility to create 2 logical volumes. ```apps-lv``` (Use half of the PV size), and ```logs-lv``` Use the remaining space of the PV size. ```sudo lvcreate -n apps-lv -L 14G webdata-vg``` and ```sudo lvcreate -n logs-lv -L 14G webdata-vg``` NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
- Verify that your Logical Volume has been created successfully by running sudo lvs
- <img width="1010" alt="image" src="https://user-images.githubusercontent.com/29708657/228915680-8c5ffe08-7876-4723-a02b-7fdecb593d85.png">
- Verify the entire setup: ```sudo vgdisplay -v #view complete setup - VG, PV, and LV``` and ```sudo lsblk```
- <img width="530" alt="image" src="https://user-images.githubusercontent.com/29708657/228916768-6d0ccdf5-9d17-4671-9aa1-eea89d3c3943.png">
and
- <img width="743" alt="image" src="https://user-images.githubusercontent.com/29708657/228916335-2dc431ed-71c7-4291-985f-a2e671eef237.png">
- Use mkfs.ext4 to format the logical volumes with ext4 filesystem: ```sudo mkfs -t ext4 /dev/webdata-vg/apps-lv``` and ```sudo mkfs -t ext4 /dev/webdata-vg/logs-lv```
- <img width="941" alt="image" src="https://user-images.githubusercontent.com/29708657/228917396-84d62a01-2d3a-4e06-bb34-9b73cc7208cb.png">

