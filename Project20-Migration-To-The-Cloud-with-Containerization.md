# MIGRATION TO THE СLOUD WITH CONTAINERIZATION. PART 1 – DOCKER & DOCKER COMPOSE
> What is Docker? Docker is a platform designed to help developers build, share, and run container applications. Docker provides the ability to package and run an application in a loosely isolated environment called a container.

**Install Docker and prepare for migration to the Cloud**
First, we need to install Docker Engine, which is a client-server application that contains:
  -  A server with a long-running daemon process dockerd.
  -  APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
  -  A command-line interface (CLI) client docker.

-----
<img width="443" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/d068721a-756f-4269-b31f-72ce6888b82e">

Step 1: Pull MySQL Docker Image from Docker Hub Registry: `sudo docker pull mysql:latest`
 - If you are interested in a particular version of MySQL, replace latest with the version number. Visit Docker Hub to check other tags
 - Verify the image: `sudo docker image ls`

-----
<img width="850" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/686305c0-788c-4c79-b487-66717d7dca06">

Step 2: Deploy the MySQL Container to your Docker Engine
  *  Once you have the image, move on to deploying a new MySQL container with: `docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql:latest`
      -  Replace <container_name> with the name of your choice. If you do not provide a name, Docker will generate a random one
      -  The -d option instructs Docker to run the container as a service in the background
      -  Replace <my-secret-pw> with your chosen password
      -  In the command above, we used the latest version tag. This tag may differ according to the image you downloaded
  *  Then, check to see if the MySQL container is running: Assuming the container name specified is MySQL: `docker ps -a `
-----
<img width="1053" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/185afbfb-afe6-4682-a78d-5b16cb64555a">


### CONNECTING TO THE MYSQL DOCKER CONTAINER

Step 3: Connecting to the MySQL Docker Container
We can either connect directly to the container running the MySQL server or use a second container as a MySQL client.
**Approach 1**
*Connecting directly to the container running the MySQL server:*
-----
Two approaches to connect to the container as shown in the image are:
  1. Run the command to enter the container shell first, then login to the mysql server when you are already logged in to the container shell. Enter password when prompted. It has to be the passwoord we set earlier: `sudo docker exec -it mysqldb bash`
  2. Run the entire command on one line and select mysql as the shell you want to log in directly to. Also enter the password when prompted: `sudo docker exec -it mysqldb mysql -p` . This can also work to specify the user: `sudo docker exec -it mysqldb mysql -p -u root`
<img width="808" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/d7bc3cd2-e7d2-4296-aa96-1171b3d66fac">

-----
Change the server root password to protect your database. Exit the the shell with exit command
**Flags used**
  -  `exec` used to execute a command from bash itself
  -  `-it` makes the execution interactive and allocate a pseudo-TTY
  -  `bash` this is a unix shell and its used as an entry-point to interact with our container
  -  `mysqldb` is the container name used earlier.  The second mysql in the command “docker exec -it mysqldb mysql -u root -p” serves as the entry point to interact with mysql container just like bash or sh
  -  `-u` mysql username
  -  `p` mysql password

**Approach 2**
*At this stage you are now able to create a docker container but we will need to add a network. So, stop and remove the previous mysql docker container.*
```
sudo docker ps -a
sudo docker stop mysqldb 
sudo docker rm mysqldb or <container ID> 04a34f46fb98
```
-----
<img width="1129" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/3bee0252-4581-4394-9d32-f1cb178b70a7">
-----

**First, create a network
For clarity’s sake, we will create a network with a subnet dedicated to our project and use it for both MySQL and the application so that they can connect: `docker network create --subnet=172.18.0.0/24 tooling_app_network`

-----
<img width="1129" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/633e7c01-4664-4d9f-b447-b77bed74a822">

*Run the MySQL Server container using the created network.*
-  First, let us create an environment variable to store the root password: `export $MYSQL_PW=<value>`
-  Verify the environment variable is created: `echo $MYSQL_PW`
-  Then, pull the image and run the container, all in one command like below: `sudo docker run --network tooling_app_network -h mysqldbhost --name=mysqldb -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql:latest`

**Flags used**
  -  `-d` runs the container in detached mode
  -  `--network` connects a container to a network
  -  `-h` specifies a hostname

If the image is not found locally, it will be downloaded from the registry.
Verify the container is running: `sudo docker ps -a`

-----
<img width="1349" alt="image" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/49ac1ba5-4e68-43f1-9754-e60671ec7c90">
-----

