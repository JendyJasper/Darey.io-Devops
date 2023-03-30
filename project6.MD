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
